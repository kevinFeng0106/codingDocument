# 在Linux中使用Docker

在网上看到很多视频讲`docker`都是跑在`vmware`中的

而这篇文档就讲一下在`wsl`中安装`linux`发行版使用`docker`



### 一、安装WSL

> `wsl`是`windows`下的`linux`子系统，安装简便，开箱即用



##### 1. 安装方式一

第一种方式就是使用管理员权限，在`powershell`中输入如下命令：

```sh
wsl install
```

安装方式非常简单，但是这样安装，经常会莫名其妙地停掉，所以不推荐这种安装方式



##### 2. 安装方式二

第二种方式是我们手动操作

* 首先在`windows`中搜索**”关闭或启用功能“**：

![](http://ldmblog.ifoodin.com/20230916095903.png)



* 然后勾选上**“适用于windows的linux子系统”**和**“虚拟机平台”**两项：

![](http://ldmblog.ifoodin.com/20230916100134.png)



* 安装完成后重启，然后再微软商店中就可以选择`linux`发行版，随便选择你喜欢的就好：

![](http://ldmblog.ifoodin.com/20230916100608.png)





### 二、启动WSL

> 启动`wsl`直接以管理员身份打开`powershell`，然后输入命令即可



##### 1. 查看WSL的启动状态及版本

可以发现现在`wsl`尚未启动，并且版本为`wsl2`：

![](http://ldmblog.ifoodin.com/20230916100939.png)

我们输入如下命令即可直接启动：

```sh
# 在用户的家目录启动
wsl ~
```

然后便会进入`ubuntu`系统：

![image-20230916101239392](C:\Users\14180\AppData\Roaming\Typora\typora-user-images\image-20230916101239392.png)



##### 2. 然后打开Windows的资源管理器

打开文件夹我们可以看见有一个`Linux`的选项，这就是我们安装的`wsl`，里面可以查看你的`linux`发行版的目录和文件：

![](http://ldmblog.ifoodin.com/20230916101629.png)





### 三、Linux中安装Docker

> 在`wsl`中微软官方文档给出的`docker`安装方法是安装`Desketop Docker for WIndows`，但是我们这里直接安装原生`docker`



##### 1. 首先安装一些基础软件包

* ca-certificates - 提供 CA 证书的 Bundle 

* curl - 命令行下的 HTTP 请求库 

* gnupg - GnuPG 加密验证工具 

* lsb-release - 提供 Linux 系统发行版信息的工具

```sh
sudo apt-get install ca-certificates curl gnupg lsb-release
```



##### 2. 安装官方的GPG key

`GPG key`主要用于验证下载软件包的完整性和来源合法性

```sh
# 设定好我们要存储的目录
sudo mkdir -p /etc/apt/keyrings
# 下载docker的gpg公钥，解密并保存
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```



##### 3. 将Docker添加到apt资源列表并更新

将`docker`软件源添加到`apt`资源列表：

```sh
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | 
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

更新`apt`：

```sh
# 系统会自动从docker源下载索引
sudo apt update
```



##### 4. 下载Docker CE

`docker ce`是社区版，免费的：

```sh
sudo apt install docker-ce
```





### 四、简单使用Docker

> `docker`的核心组成就是镜像和容器，我们需要拉取镜像，然后用镜像创建容器，再运行容器，我们每创建一个容器就相当于得到了一台虚拟主机，但是这些虚拟主机都是共用同一个宿主机环境的



##### 1. 拉取镜像

这里拉取官方的入门镜像`hello-world`：

```sh
docker pull hello-world
```

然后我们可以输入如下命令查看`docker`中已经安装的镜像：

```sh
docker images
```

![](http://ldmblog.ifoodin.com/20230916110623.png)



##### 2. 创建并运行容器

我们直接输入如下命令，就可以创建一个新的容器：

```sh
docker run hello-world
```

如果是已有的容器，可以用如下命令：

```sh
docker start <容器名> 
# 或者： 
docker start -i <容器名>
```

只要看到如下信息，就说明我们`hello-world`镜像创建的容器正常运行了：

![](http://ldmblog.ifoodin.com/20230916111255.png)