> 打算研究一下网站中接入第三方登录，但是QQ和微信要审核备案，挺麻烦的，所以就用github了，开箱即用，很方便；本篇博客如有错误，欢迎指出



### 关于Oauth

> 关于oauth，需要先说明一下几个概念：
> 1. 重定向（redirect）：是指重定向到第三方服务商的授权页面
> 2. 授权回调（callback）：是指用用户确认授权后回调到的url地址，一般是一个后端接口

*第三方登录需要用到`oauth`授权，其大概的流程是：*

1. 用户点击第三方登录按钮后，**重定向**（redirect）第三方服务商授权页面
* 跳转到授权页面的时候需要携带指定的参数，不同的服务商有区别，具体可以查阅官方的open文档

2. 用户确定授权后，**授权回调**（callback）到指定的url
* 这一步服务商会携带`code`，也就是授权码，作为`query`参数拼接到url后面
* 这个url实际上就是**后端的一个接口**，后端通过`code`来请求到`token`
* 这个过程中，**前端是没有页面显示的**，然后后端得到token后，保存好，再通知前端**重定向**到指定的网站页面

---

### github第三方登录具体实现

##### 在github的`Open auth`中注册一个应用

* 填写`应用名字`、`主页网址`、`授权回调地址`，其他的填不填都行
* 然后复制github给的密钥，一定要复制！不然以后就看不到了

![](http://ldmblog.ifoodin.com/20230720111703.png)


##### Vue中的配置

* html结构
![](http://ldmblog.ifoodin.com/20230720112310.png)

* 将几个主要参数作为query参数
![](http://ldmblog.ifoodin.com/20230720112152.png)

* 然后成功跳转到授权页面
![](http://ldmblog.ifoodin.com/a37425cb0a45720b5cc0346739c46b5e.png)

##### 用户点击确认授权后的逻辑

> 这里仅做演示，发现能拿到token即可，并没有真正的后端处理逻辑

* 回调到你指定的授权回调地址，url后面就会附上`code`，即授权码
*PS：我因为懒得写API，就直接回调到前端页面了*
![](http://ldmblog.ifoodin.com/20230720112547.png)

* 拿到`code`后，去`apiFox`用`code`和其他的参数请求`token`
![](http://ldmblog.ifoodin.com/20230720101438.png)

* 然后就可以看到服务商返回的`token`了
*PS：`code`是有过期时间的*

  过期显示如下：
  ![](http://ldmblog.ifoodin.com/20230720101244.png)

  成功显示如下：
  ![](http://ldmblog.ifoodin.com/20230720101351.png)

*然后我们就可以用`token`结合`client_id`和`client_secret`来请求用户信息了*