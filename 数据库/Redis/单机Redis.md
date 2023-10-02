# 单机Redis

`redis`是很流行的`nosql`数据库，以键值对的形式来进行数据存储，数据写在内存中，适合作为缓存



### 一、Redis实例的启动

每个`redis`实例的启动都需要占用一个端口

我们在`cmd`或者`pwershell`中输入如下命令：

```bash
# 使用自定义配置文件来启动
redis-server.exe redis.windows.conf
```



### 二、SpringBoot中操作Redis

因为这只是一个简单的`redis`实例，所以我们就用`spring cache`来简化`redis`的操作，这样我们就可以专注于数据库的业务逻辑，而不需再另外编写`redis`的逻辑，`spring cache`会自动帮我们做好缓存



##### 1. 引入依赖

引入缓存、连接池和`redis`的依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```



##### 2. 在yml配置文件中添加Redis信息

设定好`redis`实例的地址，`type`选择`redis`：

```yaml
spring:
  redis:
    database: 0
    host: localhost
    port: 6379
  cache:
    type: redis
```





##### 3. 开启缓存

在你的启动类中开启`@EnableCaching`：

```java
@SpringBootApplication
@EnableCaching
public class SpringRedisDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringRedisDemoApplication.class, args);
    }
}
```



##### 4. 编写Redis配置类

`spring cache`为我们准备了多种`CacheManager`，其中就包括`RedisCacheManager`，想要什么缓存管理器都可以自定义

同时我们还可以自定义`KeyGenerator`，来减轻我们定义`key`的困扰

```java
@Configuration
public class RedisConfiguration {

    @Autowired
    LettuceConnectionFactory lettuceConnectionFactory;

    // 自定义redis缓存管理器
    @Bean
    public RedisCacheManager redisCacheManager() {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
                .serializeValuesWith(RedisSerializationContext
                        .SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));
        
        RedisCacheManager build = RedisCacheManager.builder(lettuceConnectionFactory)
                .cacheDefaults(config)
                .build();

        return build;
    }

    // TODO 配置KeyGenerator
}
```



##### 5. 在service中使用spring cache

注入`RedisCacheManager`后就可以在方法上**注解**使用缓存管理器来简化`redis`缓存操作了：

```java
@Service
public class UserService {

    @Autowired
    RedisCacheManager redisCacheManager;

    @Autowired
    UserMapper userMapper;

    /**
    * 指定缓存空间和缓存管理器
    * 如果后续自定义了KeyGenerator也可以注入使用
    */
    @Cacheable(value = "userInfo2", key = "#myUser.account", cacheManager = "redisCacheManager")
    public MyUser finUser(MyUser myUser){
        System.out.println("第二次没有这一句的话就说明直接读取缓存");
        return userMapper.findUser(myUser);
    }
}
```

