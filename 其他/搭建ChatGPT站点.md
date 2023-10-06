# 搭建ChatGPT站点

在B站上看见了一个很有意思的搭建`ChatGPT`站点的视频，于是想动手试一试，这篇文档就记录一下搭建的过程和问题

该站点基于`github`上的一个开源项目，网址：[Chanzhaoyu/chatgpt-web: 用 Express 和 Vue3 搭建的 ChatGPT 演示网页 (github.com)](https://github.com/Chanzhaoyu/chatgpt-web)

如何搭建参考这篇文档：[ChatGPT搭建教程 - Powered by MinDoc (muluhub.com)](https://doc.muluhub.com/docs/chatgpt/chatgpt-1f1usj6gl3bit#2pvk6a)





### 一、云服务器配置

毫无疑问，我们需要一台云服务器，然后推荐大家使用这个“香草云”来购买：https://www.xiangcaoyun.com

确实非常的便宜，比腾讯阿里便宜太多太多了

操作系统选择了`Cent OS`，虽然是准备不维护的系统，但是不用这个系统的话搭建会有些问题



##### 1. 注册并购买香港云服务器

自己弄着玩的服务器就不要买国内的，因为备案很烦

然后充值并购买即可，我买的是`1C 2G`的，居然只要19元，太划算了



##### 2. 添加安全组

安全组就是虚拟防火墙，然后我们打开云服务器的管理面板

在“安全组”选项中添加规则，注意：

端口必须是1002，其他的端口在搭建时会有错误

授权`ip`默认是`0.0.0.0`，即监听来自所有`ip`源的访问

![](http://ldmblog.ifoodin.com/20231002110829.png)







### 二、Docker环境部署OpenAI

##### 1. 首先下载一个脚本

这是一个`docker`安装脚本，该脚本会根据操作系统自动下载和安装`Docker Engine`

输入如下命令：

```sh
curl -fsSL https://get.docker.com -o get-docker.sh
```

然后`ls`一下，查看是否下载成功

![](http://ldmblog.ifoodin.com/20231004155759.png)

然后执行这个脚本，执行后需要一些安装时间

```sh
sh get-docker.sh
```

然后启动`docker`服务

```sh
systemctl start docker
```

我们检查一下`docker`服务的运行状态，输入如下命令：

```sh
systemctl status docker
```

发现状态是`active(running)`即可

![](http://ldmblog.ifoodin.com/20231004160959.png)



##### 2. 然后把容器跑起来

我们输入如下命令

注意，这里我们端口映射宿主机的端口必须是`1002`

然后因为镜像是在线拉取的，不是本地的，所以需要等待一些时间

```sh
docker run --name chatgpt-web -d -p 1002:3002 \
--env OPENAI_API_KEY=<你的密钥> \
--env AUTH_SECRET_KEY=<设置你的密码> \
--env OPENAI_API_BASE_URL=https://api.openai.com  registry.ap-northeast-1.aliyuncs.com/generative/chatgpt-web
```



##### 3. 访问ChatGPT站点

在浏览器输入我们的云服务器的公网`ip`，加上`1002`端口，就可以访问到我们的`ChatGPT`站点了

我们输入密码即可开始`chat`^_^

![](http://ldmblog.ifoodin.com/20231004163036.png)

然后你可能会看到这样的情况

这是因为调用`openai API`是要钱的，你没有充钱或者余额不够了就会这样

你可以在这里：[Usage - OpenAI API](https://platform.openai.com/account/usage)，查看使用额度或者充钱

![](http://ldmblog.ifoodin.com/20231004163910.png)





### 三、为站点绑定域名



##### 1. 购买域名

首先购买一个域名，我是在西部数码购买的：https://www.west.cn/

![](http://ldmblog.ifoodin.com/20231005181831.png)





##### 2. 获取SSL证书

购买后，实名认证成功之后，去阿里云获取免费`SSL`证书：https://yundun.console.aliyun.com/

进入 “控制台” ，然后搜索`ssl`，选择 “ssl服务"，点击 ”购买证书“

阿里云一年可以免费领取20张`ssl`证书，领取后，我们创建一张证书

![](http://ldmblog.ifoodin.com/20231005182105.png)



##### 3. SSL证书解析

然后点击 ”证书申请“ ，我们把自己的域名加上`www`前缀后贴到里面

![](http://ldmblog.ifoodin.com/20231005182509.png)

提交审核后，我们需要回到域名购买商处配置`DNS`解析，按照对应的值复制粘贴即可

![](http://ldmblog.ifoodin.com/20231005182751.png)

然后回到阿里云验证`DNS`就可以了

![](http://ldmblog.ifoodin.com/20231005203727.png)

然后我们还需要添加域名解析的两种方式，把你的`ip`添加上去即可

需要添加两条记录，主机名分别为`@`和`www`

![](http://ldmblog.ifoodin.com/20231005204445.png)



##### 4. 安装Nginx环境

安装并启动`niginx`服务，输入如下命令：

```sh
sudo yum install nginx
sudo systemctl start nginx
# 设置服务自启动
sudo systemctl enable nginx
# 查看服务状态
sudo systemctl status nginx
```

![](http://ldmblog.ifoodin.com/20231006110227.png)



##### 5. 将SSL证书上传至云服务器

在证书成功签发后，我们可以在阿里云控制台下载证书到本地

B站的up主用的是`xftp`直接把本地的·文件拖到云服务器里面，但是我觉得那玩意太丑了

所以我选择在`git bash`中用`scp`命令将本地的`ssl`证书文件传到云服务器上

```sh
scp E:/SSL/chatgpt-web/www.lordmoon.pem root@149.88.68.81:/etc/nginx
scp E:/SSL/chatgpt-web/www.lordmoon.key root@149.88.68.81:/etc/nginx
```

然后我们能在云服务器上的`/etc/nginx`目录下看到这两个文件即可

![](http://ldmblog.ifoodin.com/20231006111119.png)

 

##### 6. 修改Nginx配置文件

先安装`vim`文本编辑器，然后进入配置文件修改

```sh
yum install vim
vim nginx.conf
```

将原本的`server{}`包裹的内容整段换成如下内容：

```sh
proxy_buffering off; 

# 配置代理群组
upstream chatgpt-web {
    server 127.0.0.1:1002 weight=1;
}

# http访问端口
server {
  listen 80;
  server_name www.替换的域名 替换的域名;
  location / {
    rewrite ^(.*)$ https://www.替换的域名; 
  }
}

# https访问端口
server {
  listen 443 ssl;
  server_name www.替换的域名;
  ssl_certificate /etc/nginx/替换的SSL证书.pem;
  ssl_certificate_key /etc/nginx/替换的SSL证书秘钥.key;
  location / {
    proxy_pass http://chatgpt-web;
  }     
}
```

随后检查一下`nginx`配置文件的语法是否正确，输入如下命令：

```sh
nginx -t
```

如果输出以下结果，说明没有问题：

![](http://ldmblog.ifoodin.com/20231006111926.png)



##### 7. 重启Nginx服务

输入如下命令重启：

```sh
systemctl restart nginx
```

如果重启还是无效，有可能是`80`端口被占用了

直接检查服务运行情况，然后`kill`掉对应的进程，再重新启动

```sh
# 查看服务与对应端口
netstat -lntp
# kill掉对应进程
kill <进程id>
```

然后就大功告成啦！！！输入域名，就可以以`https`协议访问网站了

![](http://ldmblog.ifoodin.com/20231006203132.png)