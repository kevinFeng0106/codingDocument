# 在Docker中搭建RabbitMQ集群

首先，`rabbitMQ`的实例无法像`redis`那样可以在一台主机中开启多个，所以我们只能用多机的方法

但是如果在`vmware`中开启多台虚拟机，会很吃性能

所以我们使用`wsl`，然后用`docker`来创建多个容器，以此实现多机



### 一、拉取RabbitMQ的镜像

输入如下命令即可：

```sh
docker pull rabbitmq:3.7-management
```

然后查看`images`是否存在：

![](http://ldmblog.ifoodin.com/20230916131013.png)





### 二、创建三个容器

- `-d`: 在后台运行容器
- `--hostname rabbitmq01`: 将容器主机名设置为rabbitmq01
- `--name rabbitmqCluster01`: 将容器名设置为rabbitmqCluster01
- `-v pwd/rabbitmq01:/var/lib/rabbitmq`: 将当前目录下的rabbitmq01目录挂载到容器内的/var/lib/rabbitmq,用于数据卷持久化
- `-p 15672:15672 -p 5672:5672`: 对外暴露web管理端口15672和AMQP端口5672
- `-e RABBITMQ_ERLANG_COOKIE='rabbitmqCookie'`: 设置RabbitMQ集群内通信的密码
- `rabbitmq:3.7-management`: 使用rabbitmq:3.7-management镜像



解析一下上面的参数：

设置`.erlang.cookie`这一步是必须的，因为`rabbitMQ`集群的所有结点必须要`.erlang.cookie`文件中的值一致

也有人通过进入容器后修改`/var/lib/rabbitmq/.erlang.cookie`文件来操作，但是我这样试过失败了

使用`-v pwd/rabbitmq01:/var/lib/rabbitmq`命令可以保证即使把容器删除了，然后使用相同的命令再创建一个容器时，仍然用的是以前的数据，这样可以做到数据持久化

```sh
#rabbitmqCluster01 主节点
docker run -d --hostname rabbitmq01 --name rabbitmqCluster01 -v `pwd`/rabbitmq01:/var/lib/rabbitmq -p 15672:15672 -p 5672:5672 -e RABBITMQ_ERLANG_COOKIE='rabbitmqCookie' rabbitmq:3.7-management

#rabbitmqCluster02 从节点
docker run -d --hostname rabbitmq02 --name rabbitmqCluster02 -v `pwd`/rabbitmq02:/var/lib/rabbitmq -p 7002:15672 -p 5673:5672 -e RABBITMQ_ERLANG_COOKIE='rabbitmqCookie'  --link rabbitmqCluster01:rabbitmq01 rabbitmq:3.7-management

#rabbitmqCluster03 从节点
docker run -d --hostname rabbitmq03 --name rabbitmqCluster03 -v `pwd`/rabbitmq03:/var/lib/rabbitmq -p 8002:15672 -p 5674:5672 -e RABBITMQ_ERLANG_COOKIE='rabbitmqCookie'  --link rabbitmqCluster01:rabbitmq01 --link rabbitmqCluster02:rabbitmq02  rabbitmq:3.7-management

```





### 三、搭建RabbitMQ集群

>我们以`rabbitmqCluster01`容器作为主节点，其他两个作为从节点



##### 1. 将从节点加入主节点集群

我们对两个从节点分别输入以下命令

其中`rabbit@rabbitmq01`为主节点的名称，，默认前缀是`rabbit`，后面加上主机名：

注意这里必须要`reset`，否则会报节点拒绝的错误

```sh
docker exec -it <容器名称> /bash
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl join_cluster --ram rabbit@rabbitmq01
rabbitmqctl start_app
```



##### 2. 查看RabbitMQ集群的状态

在任意节点中输入如下命令：

```sh
rabbitmqctl cluster_status
```

![](http://ldmblog.ifoodin.com/20230916000639.png)

也可以在管理页面查看：

![](http://ldmblog.ifoodin.com/20230916000720.png)



##### 3. 新建队列试验

我们在`rabbitmq01`节点中新建一个队列：

![image-20230916201732965](C:\Users\14180\AppData\Roaming\Typora\typora-user-images\image-20230916201732965.png)

可以看到在`rabbirmq02`节点中也可以看到该队列：

![image-20230916201941642](C:\Users\14180\AppData\Roaming\Typora\typora-user-images\image-20230916201941642.png)



### 四、常见异常报错处理

> 在使用`docker`时，不可避免会出现很多`bug`，这里记录一下一些我搭建`rabbitMQ`时处理的报错



##### 1. 重启节点后无法加入集群

重启节点后，可能报如下错误：

![](http://ldmblog.ifoodin.com/20230916200701.png)

这是因为当我们关闭容器时，`rabbitMQ`会在主机的`Database`自动记录缓存之前已经搭建好的集群：

![](http://ldmblog.ifoodin.com/20230916202246.png)

所以当我们使用`start`命令再次启动容器时，在`stop_app`之后，应该再`reset`一遍，然后再加入集群：

```sh
rabbitmqctl stop_app
# 重置一下
rabbitmqctl reset
rabbitmqctl join_cluster rabbit@rabbitmq01
rabbitmqctl start_app
```

