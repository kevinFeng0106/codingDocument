# RabbitMQ的使用

`rabbitMQ`是当前非常流行的一种消息队列

它的整体部分为一个`Vhost`（虚拟主机），在一个`Vhost`内，可以定义多个`exchange`和`queue`

然后`exchange`会根据`routing key`来将消息放到对应的队列中

在外部可以用`connection`和`channel`来创建**生产者**和**消费者**，连接队列后可以发布和接收消息

以下是`rabbitMQ`的结构图：

![](http://ldmblog.ifoodin.com/rabbitMQ%E7%BB%93%E6%9E%84%E5%9B%BE2.png)





### 一、RabbitMQ服务的启动和配置

##### 1. 使用cmd / powershell

必须使用`cmd`或者`powershell`，而且必须是管理员身份，否则就会报 “系统拒绝访问“ 的错误

进入到`rabbitMQ`的`sbin`目录下，然后输入如下命令：

```bash
# 启动rabbitmq服务
rabbitmq-service start
```



##### 2. 查看服务状态

我们在`sbin`目录下输入：

```bash
rabbitmqtcl status
```

这里可能会有类似如下的报错：

```bash
Error: unable to perform an operation on node 'rabbit@wangshuo'. Please see diagnostics information and suggestions below.

Most common reasons for this are:

 * Target node is unreachable (e.g. due to hostname resolution, TCP connection or firewall issues)
 * CLI tool fails to authenticate with the server (e.g. due to CLI tool's Erlang cookie not matching that of the server)
 * Target node is not running

In addition to the diagnostics info below:

 * See the CLI, clustering and networking guides on https://rabbitmq.com/documentation.html to learn more
 * Consult server logs on node rabbit@wangshuo
 * If target node is configured to use long node names, don't forget to use --longnames with CLI tools

DIAGNOSTICS
===========

attempted to contact: [rabbit@wangshuo]

rabbit@wangshuo:
  * connected to epmd (port 4369) on wangshuo
  * epmd reports: node 'rabbit' not running at all
                  no other nodes on wangshuo
  * suggestion: start the node

Current node details:
 * node name: 'rabbitmqcli-19760-rabbit@wangshuo'
 * effective user's home directory: C:\Users\13343
 * Erlang cookie hash: y1wQRjvcOXX+x5pqGKKOWw==
```

这是因为`rabbitMQ`的语言环境是`ErLang`，而`ErLang`会生成两个`cookie`文件：

![](http://ldmblog.ifoodin.com/20230913211712.png)

我们需要将第一个路径中的，也就是`Users`目录下的`.erlang.cookie`中的内容替换为另一个目录下的，然后再输入以下命令重启`rabbitMQ`服务就可以正常查看`rabbitMQ`服务的状态了：

```bash
# 先停止，再启动
rabbitmq-service stop 
rabbitmq-service start
```

然后我们就可以访问`localhost:15672`来查看`rabbitMQ`的`web management`页面



### 二、RabbitMQ的基本结构搭建

> 搭建`rabbitMQ`需要先有一个`Vhost`，然后`Vhost`需要有对应的`user`，然后再进行内部的`exchange`和`queue`的搭建



##### 1. 新建一个User

这里我们命名该`user`为`test`

在`admin`的`Users`栏新建`user`，设置好名字和密码，赋予其`Admin`的权限：

![](http://ldmblog.ifoodin.com/20230913150456.png)



##### 2. 新建一个Vhost

这里我们命名这个`Vhost`为`/test`，它是一个路径格式的：

![](http://ldmblog.ifoodin.com/20230913150625.png)

新建`Vhost`后，`rabbitMQ`会自动为我们在该`Vhost`中新增默认的几个交换机：

![](http://ldmblog.ifoodin.com/20230913213135.png)



##### 3. 给User添加Vhost的访问权限

在`Users`栏中点击对应的`user`，然后添加对应`Vhost`的`Permissions`：

![](http://ldmblog.ifoodin.com/20230913150709.png)



##### 4. 添加一个经典队列

选中`Queues`，然后在`Add a new queue`处，设置好对应的`Vhost`，类型选择`Classic`，不需要加参数：

![](http://ldmblog.ifoodin.com/20230913155809.png)



##### 5. 给队列绑定交换机

在队列的`Bindings`选项中，我们设置好`exchange`的名字和`Routing Key`，来给队列绑定交换机：

![](http://ldmblog.ifoodin.com/20230913190618.png)






### 三、使用RabbitMQ发布和接收消息

> 之前我们都是简单地在`web management`页面来使用`direct`类型的交换机来进行操作，现在我们就在`spring boot`中使用`rabbitMQ`来发布和接收消息



##### 1. 前置工作

* 首先需要引入依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

* 在`yml`配置文件中设定`rabbirMQ`的配置：

```yaml
spring:
  # rabbitMQ基本信息
  rabbitmq:
    addresses: 127.0.0.1
    username: test
    password: test123
    virtual-host: /test
    # 这个配置表示配置为消息手动响应
    listener:
      simple:
        acknowledge-mode: manual
```



##### 2. 使用direct类型的交换机

`direct`是直连类型的交换机，一个交换机能与多个队列绑定，但是**发布一次消息只能发送到一个队列**



* 创建`rabbieMQ`配置类：

```java
@Configuration
public class RabbitConfiguration {
	// 配置交换机
    @Bean("directExchange")
    public Exchange exchange() {
        // 交换机类型为direct
        return ExchangeBuilder.directExchange("amq.direct").build();
    }

    // 创建队列
    @Bean("testQueue")
    public Queue queue() {
        return QueueBuilder
                .nonDurable("my-test-queue")
                .build();
    }

    // 给队列绑定交换机
    @Bean("binding")
    public Binding binding(@Qualifier("directExchange") Exchange directExchange,
                           @Qualifier("testQueue") Queue queue) {
        return BindingBuilder
                .bind(queue) // 绑定队列
                .to(directExchange) // 指定交换机
                .with("my-test-key") // 设置routing-key
                .noargs();
    }
}
```



* 创建一个消费者：

```java
// 该消费者需要注册为bean
@Component
@Slf4j
public class RabbitConsumer {

    @RabbitListener(queues = "my-dl-queue", messageConverter = "jacksonConverter")
    public void testListener(Channel channel, Message message) {
        System.out.println(new String(message.getBody()));
        try {
            // 这里需要手动ack消息
            // 如果没有上面的配置的话，就会自动ack
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
            System.out.println("接收成功！");
        } catch (Exception e) {
            log.error(e.getMessage());
        }
    }
}
```



* 在`controller`中使用生产者：

```java
@RestController
@Slf4j
public class TestController {

    // 注入生产者依赖
    @Resource
    RabbitTemplate rabbitTemplate;

    @GetMapping("/test")
    public String test() {
        try {
            // 发布消息
            rabbitTemplate.convertAndSend("amq.direct", "my-test-key", new TestUser());
            return "success";
        } catch (Exception e) {
            log.error(e.getMessage());
            return "fail";
        }
    }
}
```





##### 3. 创建死信队列

死信队列其实也是一个基本的队列，并不是`rabbitMQ`帮我们封装好的

死信队列的作用是接收别的`queue`中的过期的或超出容量限制的消息

我们在设置死信队列时，可以设置`maxLength`和`ttl`来让特定条件下的消息成为死信



* 首先在`rabbitMQ`配置类中添加死信队列的配置，创建的方式与一般的队列一样：

```java
// 创建死信交换机
@Bean("dlExchange")
public Exchange dlExchange() {
    return ExchangeBuilder.directExchange("dlx.exchange").build();
}

// 新建一个死信队列
@Bean("dlQueue")
public Queue dlQueue() {
    return QueueBuilder.nonDurable("my-dl-queue").build();
}

// 死信队列绑定
@Bean("dlBinding")
public Binding dlBinding(@Qualifier("dlExchange") Exchange dlExchange,
                         @Qualifier("dlQueue") Queue dlQueue) {
    return BindingBuilder
        .bind(dlQueue)
        .to(dlExchange)
        .with("my-dl-key")
        .noargs();
}
```



* 然后在`rabbitMQ`配置类中修改一般`Queue`的配置即可：

```java
// 创建队列
@Bean("testQueue")
public Queue queue() {
    return QueueBuilder
        .nonDurable("my-test-queue")
        .deadLetterExchange("dlx.exchange") // 指定死信队列
        .deadLetterRoutingKey("my-dl-key") // 指定死信routing-key
        .build();
}
```



##### 4 . 当传输的message为对象时

当消息为对象类型的数据时，我们可以在`rabbitMQ`的配置类中添加如下的序列化配置：

```java
// 对象序列化
@Bean("jacksonConverter")
public Jackson2JsonMessageConverter converter() {
    return new Jackson2JsonMessageConverter();
}
```



##### 5. 工作队列模式

工作队列模式，就是给一个队列设置多个监听器（消费者）：

```java
@RabbitListener(queues = "my-dl-queue", messageConverter = "jacksonConverter")
public void testListener("消费者一号监听消息：" + Channel channel, Message message) {
    System.out.println(new String(message.getBody()));
    channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
}

@RabbitListener(queues = "my-dl-queue", messageConverter = "jacksonConverter")
public void testListener("消费者二号监听消息：" + Channel channel, Message message) {
    System.out.println(new String(message.getBody()));
    channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
}
```



##### 6. 路由模式

路由模式就是使用`direct`类型的交换机，然后根据`routing key`来路由到不同的队列，这里不多赘述



##### 7. 使用fanout类型的交换机

假如我们想让多个队列绑定同一台交换机，并且需要发布一次消息就可以发送到多个`queue`中，那么就需要用到`fanout`类型的交换机

使用这种交换机的`rabbitMQ`的模式也叫做**发布订阅模式**



##### 8. 使用topic类型的交换机

`topic`类型的交换机其实就是它的`routing key`是正则的模糊匹配

也就是说可以匹配多个`routing key`，然后发送到多个队列