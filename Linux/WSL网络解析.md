# WSL网络解析

> 之前用`wsl`的时候都没太关注过它的网络配置，实际上它的网络配置是和`vaware`还有云服务器这种是有有差异的，这篇文档就来简单地总结一下



### 一、先决条件

首先这篇文档是基于`wsl2`的，与`wsl1`不一样，`wsl2`的本质就是虚拟机

所以`wsl2`会与`windows`组成一个局域网，对于这个虚拟网络，`windows`会再分配一个网络适配器





### 二、在Windows中查看WSL体系的网络配置

我们打开`cmd`，输入`ipconfig/all`查看网络配置

我们找到 “以太网适配器 vEthernet (WSL)” 一项，这就是`wsl2-windows`体系的网络配置

这里的`ipv4地址`就是在这个虚拟网络中的`windows`的`ip`地址，然后我们也可以看到这张虚拟网卡（网络适配器）的`mac`地址

![](http://ldmblog.ifoodin.com/20231003194405.png)





### 三、在WSL中查看自身的网络配置

输入`wsl ~`启动`wsl`，然后输入`ifconfig`查看`wsl`的网络配置

我们可以看到有三个配置，分别来解析一下

第一个是`docker`的网桥，这里不是重点，我有一篇文档是专门讲`docker`的，所以这里就不展开了

第三个就是本地回环了，没什么好说的

重点是第二个`eth0`，这就是`wsl`的主机`ip`了

可以发现，这个`ip`跟在`windows cmd`里面查看的不一样

这是因为这个`ip`是`wsl`的，而在`windows cmd`里面查看的`ip`是`windows`的，不要搞混了

![](http://ldmblog.ifoodin.com/20231003195431.png)



### 四、WSL与外部网络的通信

在`wsl`中，它的`DNS`配置文件存在`/etc/resolv.conf`路径下，我们`cat`出来看看

可以看到一个`ip`地址，这个地址和我们在`windows`中`ipconfig`得到的的`ipv4`地址是一样的

这就说明了`wsl`是靠`windows`来寻找真正的`DNS`解析服务的，也就是说`wsl`实际上是借助`windows`来寻找外部网络服务的

![](http://ldmblog.ifoodin.com/20231003200406.png)

但是有一种情况比较特殊，就是在`windows`的浏览器里面访问`127.0.0.1`并带上对应的端口号时，可以访问到`wsl`的`linux`发行版运行的服务