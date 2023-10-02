# 使用uni-ai进行二次开发

对于`uni-ai`及`uni-chat-ai`，官方文档讲的很少，于是我想写一篇文档整理总结一下

首先，按我的理解，我认为`uni-ai`是一个 “使用`unicloud`的`ai`功能进行开发” 的统称

它依赖于一个叫`uni-cloud-ai`的**云函数扩展库**，我们导入该扩展库即可使用`uni-ai`的功能

这篇文档先会整理一下基础的知识，然后讨论一下使用`uni-chat-ai`开源模板进行二次开发



### 一、在云函数中加载uni-cloud-ai

在云函数中使用`uni-ai`功能，需要我们先加载外部扩展库`uni-cloud-ai`的依赖，我们右键某一个云函数的文件夹即可加载

在`uni-chat-ai`的模板中，官方已经为我们配置好了

![](http://ldmblog.ifoodin.com/20230923152307.png)

查看`uni-ai-chat`云对象的`package.json`文件，我们可以发现有对应的`uni-cloud-ai`扩展依赖

![](http://ldmblog.ifoodin.com/20230923152429.png)





### 二、uni-ai聊天功能的基础使用

> 这里主要有侧重点地讲一下`uni-ai`的`stream`响应还有一些需要封装的地方，其他的东西就简略带过了



 ##### 1. 获取LLM实例

这个就是向`ai`发送消息所必需的，基本的使用可以参考官方文档：[获取LLM实例](https://zh.uniapp.dcloud.io/uniCloud/uni-ai.html#get-llm-manager)



然后主要讲一下它的`messages`数组里面的`role`，`uni-ai`有三个`role`

* `system`是给开发者用于限制`ai`回答问题的范围的
* `user`一般用于用户打字询问的
* `assitant`就是`ai`返回的内容，当我们使用该角色发送消息时，一般是用于联系上下文，那么问题就来了，我们自己去手动封装联系上下文，那不是太麻烦了？所以`uni-chat-ai`已经帮我们封装好了，我们后面讲



##### 2. stream响应

我们使用`gpt`的时候经常看到它会一个字一个字地回复，这就是`stream`响应

我们来看看`uni-ai`这里面是如何使用的，可以先看看文档了解一下：[流式响应](https://zh.uniapp.dcloud.io/uniCloud/uni-ai.html#chat-completion-stream)



在`stream`响应中，需要云函数/云对象支持`sseChannel`，而`sseChannel`又是基于`uni-push`的，所以我们应该先开启`uni-push`服务

具体请参考官方文档：[uni-push2统一推送 | uni-app官网 (dcloud.io)](https://zh.uniapp.dcloud.io/unipush-v2.html#第四步-服务端推送消息)



在云函数中使用`sseChannel`的话，客户端也必须使用`sseChannel`来接收消息

具体的使用是在客户端调用`uniCloud.SSEChannel()`来创建一个管道，然后发送给云函数，在云函数内部使用该管道向客户端发送消息

* 客户端

```js
// 先创建通道
const channel = new uniCloud.SSEChannel()

// 然后打开通道，一定要打开通道再向云函数传送
await channel.open()

// 接收云函数管道发送来的消息
channel.on('message', (message) => {
    console.log('收到消息：' + message)
})

// 然后哦调用云函数的时候作为参数传入
uni.callFunction({
    name: 'functionName'
    data:{
    	channel: channel
	}
})
```

* 云函数中

```js
exports.main = async (event, context) => {
    // 反序列化管道
    const sseChannel = uniCloud.deserializeSSEChannel(event.channel)
    // chatCompletion返回stream对象
    let streamRes = await llmManager.chatCompletion({
      messages: [{
        role: 'user',
        content: '发送消息'
      }],
      tokensToGenerate: 400,
      stream: true // 开启流式返回
    })
    ...
    return Promise((resolve, reject) => {
        // 监听stream对象的消息
        streamRes.on('message', async (message) => {
            // 向管道中写入消息
            await channel.write(message)
        })
        
        // 结束
        streamRes.on('end', async () => {
            await sseChannel.end({ 
                errCode: 0,
                errMsg: ''
            })
        })
        
        resolve({
            errCode: 0,
            errMsg: ''
        })
    })
}
```

具体可以参考官方文档：[云函数请求中的中间状态通知通道 | uni-app官网 (dcloud.io)](https://zh.uniapp.dcloud.io/uniCloud/sse-channel.html#create-sse-channel)、[SSE管道 | uni-app官网 (dcloud.io)](https://zh.uniapp.dcloud.io/uniCloud/uni-ai.html#chat-completion-stream)





### 三、uni-chat-ai的封装讲解

