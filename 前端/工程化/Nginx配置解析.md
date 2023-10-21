# Nginx配置解析

这篇文档主要就是总结一下如何配置`nginx`，至于那些有关`nginx`的作用介绍什么的，就不提了

只要知道`nginx`是一个`web`服务器/反向代理服务器就行了，详情参考这两篇博客：

[万字总结，体系化带你全面认识 Nginx ！ - 掘金 (juejin.cn)](https://juejin.cn/post/6942607113118023710#heading-6)

[Nginx 入门（一）Nginx 配置Web服务器 - 简书 (jianshu.com)](https://www.jianshu.com/p/734ef8e5a712)

这篇文档会随着自己学习不断更新



在`nginx`配置文件中，常用的配置项有如下这些：

* `main` 全局配置，对全局生效；

* `events` 配置影响 `Nginx` 服务器与用户的网络连接；

* `http` 配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置；

* `server` 配置虚拟主机的相关参数，一个 `http` 块中可以有多个 `server` 块；

* `location` 用于配置匹配的 `uri` ；

* `upstream` 配置后端服务器具体地址，负载均衡配置不可或缺的部分；

然后放一张`nginx`的层级结构图，方便理解：

![](http://ldmblog.ifoodin.com/20231016083531.png)





### 一、全局配置块

```nginx
# 定义单进程，通常将其设成CPU的个数或者内核数
worker_processes  1;

# Nginx的错误日志存放目录
error_log  /var/log/nginx/error.log warn;   
```





### 二、events块

```nginx
events {
    # 通过worker_connections和worker_processes计算maxclients
    # max_clients = worker_processes * worker_connections
    # Linux系统默认每个进程允许打开的文件描述符是1024个
    # 最大连接数max_clients指的是nginx服务器能够同时接受的客户端TCP连接数
    worker_connections  1024;
}
```





### 三、http块

这是最常用的配置块，里面通常会嵌套多层配置结构

```nginx
http {
    # 设置保持客户端连接时间，默认为75秒
	keepalive_timeout  65;
    
    # 可以设置多个server
    server {
        # 监听的端口
        listen  80;
        
        # 定义主机的名字，一般是访问的域名，可定义多个，用空格隔开
        server_name  lordmoon.site www.lordmooon.site;
        
        # 反向代理群组
        upstream mysvr {   
          server 127.0.0.1:7878;
          server 192.168.10.121:3333;  
    	}
        
        # location模块可以配置nginx如何反应资源请求
        loaction / {
        	# 网站根目录
            root  html;

            # 网站首页
            index  index.html index.htm;
    	}
       
        # 这里针对以/api为前缀的路径进行反向代理
        # 这样就可以对项目中的axios做跨域代理了
        location /api {
            proxy_pass  http://mysvr;
        }
        
        # 这样写可以精确匹配路径
        location = /test {
            proxy_pass  http://mysvr;
        }
    }
}
```

在`http`块中，我们还可以配置`https`协议对应的`ssl`证书，这和配置`http`协议的80端口配置不同：

```nginx
http {
    server {
        # https 443端口
        listen  443 ssl;
        server_name  www.lordmoon.site lordmoon.site;
        
        upstream mysvr {   
          server 127.0.0.1:7878;
          server 192.168.10.121:3333;  
    	}
        
        # 配置你下载的ssl证书
        ssl_certificate  ssl/server.pem;
        ssl_certificate_key  ssl/server.key;
        
        location /api {
            proxy_pass  http://mysvr;
        }
    }
    
    server {
        listen  80;
        server_name  www.lordmoon.site lordmoon.site;
        
        # 将请求重定向为https
        rewrite ^(.*)$ https://lordmoon.site;
    }
}
```

