# Redis集群的两种实现方式

`redis`集群的实现都依赖主从复制，所谓的主从复制，就是主节点和从节点，主节点主要负责写入数据，然后**同步到**从节点，而从节点负责读取数据



### 一、哨兵模式

> 哨兵模式是一种单`master`模式，也就是说只有一个主节点



##### 1. 首先设置好主从节点

我们先规定好所有的节点的`ip`均为`127.0.0.1`，主节点的端口为`6001`，剩下两个从节点的端口为`6002`和`6003`

在`redis.windows.conf`中配置主从节点的关系，在两个从节点的配置文件中加上：

```bash
# 这个配置表示该节点是哪个节点的从节点
slaveof 127.0.0.1 6001
```



##### 2. 然后配置哨兵

每个哨兵需要再另外启动一个`redis`实例，这里我们就只用一个哨兵，规定端口号为`7001`

在哨兵的`redis.windows.conf`文件中加上以下配置：

```bash
# 这个配置表示哨兵监控的主节点是哪一个，名字随便写都行
# 最后的1表示只要有一个哨兵认为主节点挂掉了，那么主节点就挂掉了
sentinel monitor <your-master-name> 127.0.0.1 6001 1
```



* 然后基本上哨兵模式的流程就是这样的，可以跑起来，然后**关闭主节点**的实例看看，哨兵是否会重新分配主节点

* 因为哨兵模式是单`master`的集群，所以当写入多的时候，压力会比较大，所以下面就讲多`master`的模式



### 二、Cluster模式

> 为了解决单`master`的不足，可以使用`Cluster`模式，这是一种多`master`的`redis`集群方案



##### 1. 首先在各配置文件中配置好节点

`redis`的`Cluster`模式不需要我们再去配置主从节点了，它会自动为我们分配好的

我们使用三个主节点+三个从节点的配置，规定主节点的端口为`6001`、`6002`、`6003`；从节点端口为`7001`、`7002`、`7003`

然后在每个`redis.windows-conf`配置文件中加上（或者把原本的注释打开）：

```bash
# 开启集群模式
cluster-enabled yes 
```

然后分别启动每个`redis`实例：

```bash
redis-server.exe redis.windows.conf
```



##### 2. 在客户端声明使用Cluster模式

启动客户端时携带如下参数：

我这里是为了方便看得更清晰，实际上自己在`cmd`敲的时候是不能手动换行的

```bash
# 表示创建集群
redis-cli.exe --cluster create 
# 输入所有的集群节点
127.0.0.1:6001 127.0.0.1:6002 127.0.0.1:6003 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003
# 这里表示为每个主节点分配1个从节点
--cluster-replicas 1 
```



然后我们可以看到`redis`自动为我们分配好了`Cluster`的方案：

我们只需输入`yes`即可

![](http://ldmblog.ifoodin.com/20230913102935.png)



但是当我们退出然后再次启动服务的时候，会报以下错误：

![](http://ldmblog.ifoodin.com/20230913102005.png)

这是因为`redis`集群重新启动时会读取相应备份文件的数据

所以我们需要删除所有`redis`实例中的`dump.rdb`和`nodes.conf`文件，然后重新启动，就能看到和上面一样的输出了





### 三、使用SpringBoot操作Redis集群



##### 1. 引入相关依赖

```xml
<!-- springboot中的redis依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<!-- lettuce pool 缓存连接池-->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```



##### 2. 在yml文件配置Redis相关

注意：在`springboot2`中才可以这样写，`springboot3`的`redis`配置结构不是这样的

```yaml
# 不再需要 host 和 port 配置
spring:
  redis:
    cluster:
      nodes:
        - 127.0.0.1:6001
        - 127.0.0.1:6002
        - 127.0.0.1:6003
        - 127.0.0.1:7001
        - 127.0.0.1:7002
        - 127.0.0.1:7003
    database: 0
```



##### 3. 编写相关配置类

```java
@Configuration
public class RedisConfiguration extends CachingConfigurerSupport {

    /**
    * 配置RedisTemplate用于读写
    */
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        // 设置RedisTemplate
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory);

        // 对象序列化
        GenericJackson2JsonRedisSerializer serializer = new GenericJackson2JsonRedisSerializer();
        redisTemplate.setDefaultSerializer(serializer);
        return redisTemplate;
    }

    /**
    * 自定义KeyGenerator，可保证key不为空并且不重复
    */
    @Bean
    @Override
    public KeyGenerator keyGenerator() {
        return (target, method, objects) -> {
            StringBuilder sb = new StringBuilder();
            sb.append(target.getClass().getName())
              .append(".")
              .append(method.getName())
              .append(Arrays.toString(objects));
            return sb.toString();
        };
    }
}
```



##### 4. 引入RedisTemplate并使用

然后通过`@AutoWired`引入`RedisTemplate`即可进行`set`和`get`操作：

```java
redisTemplate.opsForValue().set(key, object)
redisTemplate.opsForValue().get(key)
```

