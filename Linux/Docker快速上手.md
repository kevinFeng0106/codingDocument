# Docker快速上手

> 如何在`Linux`中安装`docker`并进行简单使用已经在另一篇文档中讲过了, 这里就不说了, 直接上手



### 一、容器网络管理

##### linux网络介绍

在了解`docker`网络之前，我们还是要先来看看`linux`主机的网络

然后我们先在宿主机环境中输入`ifconfig`查看一下网络

我们可以看到宿主机中有三个网络，下面我们分别介绍一下

![image-20231003104024784](C:\Users\14180\AppData\Roaming\Typora\typora-user-images\image-20231003104024784.png)

第一个就是`docker`网桥的网络，后面再展开说

第二个`eth0`是宿主机的`ip`地址，它既可以是公网`ip`，也可以是内网`ip`，一般我们购买的云服务器都是公网`ip`

第三个`127.0.0.1`就是本地回环地址，就不多说了



##### docker网络介绍

`docker`的默认网络有三种模式, 分别为: `bridge`, `host`, `none`: 

![](http://ldmblog.ifoodin.com/20231003104952.png)



##### bridge网络

`docker`容器默认的就是桥接网络，这个`docker0`网桥的工作方式类似于交换机

在没有手动配置的情况下, 所有的`docker`容器都会连接到这个网桥,也就是说所有`docker`容器的网络请求都会经过`docker`主机

使用`bridge`网络时，`docker0`就是`docker`容器的默认网关

`docker0`作为一个网络桥接的角色，所有`docker`容器组成它的虚拟子网



我们输入如下命令，查看`docker0`网桥的网络配置：

```sh
docker network inspect bridge
```

![](http://ldmblog.ifoodin.com/20231003205531.png)



下面放一张`bridge`网络的结构拓扑图：

![](http://ldmblog.ifoodin.com/20231003145017.png)



##### host网络

在`host`模式下，容器将不会虚拟出自己的网卡，配置自己的IP，会直接使用宿主机的`ip`地址和端口

容器与宿主机、以及其他使用`host`模式的容器可以相互通信

使用`host`模式时，如果想访问容器，我们就不需要再做端口映射了



##### none网络

顾名思义，就是什么都没有，没有`ip`，只能单机操作，任何网络都不可以访问







### 二、Docker网络实操



##### 打包一个镜像

我们先在`dicker`中拉取一个`unbuntu`的镜像: 

![](http://ldmblog.ifoodin.com/20231003102222.png)

然后我们启动`ubuntu`镜像容器: 

这里 `-it`是`-i`和`-t`的组合命令

`-i`表示以交互模式运行容器

`-t`表示为这个容器分配一个伪输入终端

```sh
docker run -it ubuntu bash
```



然后我们需要在这个`ubuntu`的容器中下载一些网络管理工具: 

```sh
apt update
apt install net-tools iputils-ping curl
```

将这个容器打包成镜像,方便后面重复使用

先看看我们的容器: 

![](http://ldmblog.ifoodin.com/20231003103420.png)

然后复制名字打包: 

我这里给这个打包的镜像命名为`ubuntu-net-study`

```sh
docker commit happy_lumiere ubuntu-net-study
```



##### 创建一个自定义网络

输入如下命令，创建我们的自定义网络，我这里命名为`test`

```sh
docker network create --driver bridge test
```

我们再查看一下这个网络：

可以看到它又添加了一个新的`bridge`网桥来作为网关

![](http://ldmblog.ifoodin.com/20231003211340.png)



##### 容器间的网络通信

`docker`容器间要网络通信，就要在同一网段，所以我们需要让容器的网络保持一致

还有我们要在一个容器中访问另一个容器，就需要通过`ip`地址来访问，但是我们并不好直接指定`ip`地址，有可能会冲突

那么我们仍然让`docker`网络自动分配`ip`地址，我们只需要指定一下`docker`容器的名字即可

这样，`docker`提供的`DNS`服务器就会为我们自动解析`docker`容器的名字

注意：这种方式只能在我们自定义的网络下才生效，默认的网络是不行的

```sh
docker run -it --name=ubuntu-test01 --network=test ubuntu-net-study
docker run -it --name=ubuntu-test02 --network=test ubuntu-net-study
```



然后这里我们启动了两个容器后，退出到宿主机中看看

这里有个小技巧，按`ctrl`+`p`+`q`，就可以保持容器的运行，然后退出到宿主机

我们可以看到，两个容器都是处于`Up`状态

![](http://ldmblog.ifoodin.com/20231003221204.png)

然后我们重新进入`ubuntu-test02`，输入如下命令：

```sh
docker attach ubuntu-test02
```

然后我们在`02`中`ping`一下`01`，发现是可以`ping`到的，说明确实在同一子网中，并且名字被成功解析

![](http://ldmblog.ifoodin.com/20231003221343.png)



除此之外，我们还可以让两个容器共享网络

在`ubuntu-test02`启动时，我们用如下方式：

```sh
docker run -it --name=ubuntu-test02 --network=container:ubuntu-test01 ubuntu-net-study
```

这样就把`ubuntu-test01`的网络共享给了`ubuntu-test02`，两个容器的网络状态就是一样的





##### 容器与外部网络通信

上面说过，`docker`网络是跟局域网类似，容器的`ip`都是私网`ip`，所有的容器都连到默认网关`docker0`上

如果我们需要访问互联网，就需要在`docker0`和宿主机之间添加一个`NAT`服务设备，`docker`已经帮我们做好了

![](http://ldmblog.ifoodin.com/20231003224527.png)

那么如果外部的网络要访问`docker`容器中的服务呢？

这个时候我们就需要设置端口映射了

下面的启动方式就是将容器的80端口绑定到了宿主机的8090端口

这样我们直接访问宿主机的8080端口，就可以直接访问到这个容器的80端口跑的服务了

```sh
docker run -d -p 8080:80 nginx
```

![](http://ldmblog.ifoodin.com/20231003225144.png)







### 三、容器的存储



##### 容器持久化存储

容器的持久化需要用到`volume`（数据卷）

我们一般的操作是将容器的数据保存到宿主机上，这样就算容器销毁，文件也可以保留

我们先创建一个`test`目录，在`test`目录中创建一个新文件，并写一些内容

```sh
mkdir test
vim test/hello.txt
```

然后在启动容器的时候将宿主机上的**目录或文件**挂载到容器的某个目录上

这里的`-v`参数就是将宿主机的`~/test`目录挂载到容器的`/root/test`目录

```sh
docker run -it -v ~/test:/root/test ubuntu-volume-study
```

此时我们进入容器查看，发现确实有这个`hello.txt`文件

当我们下次重新使用相同的命令启动容器的时候，仍然可以获得这些数据

![](http://ldmblog.ifoodin.com/20231004105743.png)



注意：如果我们启动容器时不指定宿主机的挂载路径，那么`docker`就会自动生成一个数据卷，由`docker`帮我们进行管理

这里我们仅指定容器的目录

```sh
docker run --name=ubuntu-volume-test  -it -v /root/abc ubuntu-volume-study
```

启动后，我们`ctrl`+`p`+`q`退出容器，然后`inspect`一下这个容器

我们看到`Mounts`这一项，可以发现`docker`已经为我们创建好了挂载的`volumes`目录

![](http://ldmblog.ifoodin.com/20231004112011.png)

当我们不需要使用某些数据卷的时候，可以使用如下命令删除：

```sh
# 查看数据卷名字
docker volume ls
# 删除指定数据卷
docker volume rm <数据卷名字>
```



当我们只需要复制某些文件而不需要挂载时，我们可以在宿主机中使用如下命令：

```sh
# 将宿主机的test/hello.txt文件的内容复制到容器的/root目录下
docker cp test/hello.txt <容器名>:/root
```



##### 容器间的数据共享

容器间的数据共享我们仍然用的是`volume`来实现

首先我们手动创建一个数据卷，输入如下命令，这里我给数据卷命名为`volume-test`：

```sh
docker volume create volume-test
```

然后我们启动两个容器，并指定该数据卷作为它们挂载目录

```sh
# 为其中一个容器手动指定数据卷
docker run -it -v volume-test:/root/test ubuntu-volume-study
# 为另一个容器直接使用其他容器的数据卷，挂载路径都是一样的
docker run -it --volumes-from=<其他容器名> ubuntu-volume-study
```



