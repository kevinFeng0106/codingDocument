# 基于uni-id的数据库扩展设计

最近在写一个`unicloud`的项目，里面用到了`uni-id`管理用户登录，但是它默认的数据库表的字段无法完全满足我们的业务需求

所以我刚好就以这篇文档总结归纳一下如何使用`uni-id`来进行一些二次开发和扩展设计



### 先来看看uni-id-users的表结构

这个项目使用的是微信登录，所以这里仅展示微信登录方式时存储的用户数据格式

可以看到，这里的数据是没有用户名的，所以我们应该要加上用户名

但是`uni-id`的这张表对应的业务已经是封装好了的，直接在上面改可能容易出问题

所以我们对于这种二次开发的思路是：**新开一张表，用某些数据作外键（逻辑外键），然后新增我们所需的业务字段**

```json
{
    "wx_openid": {
        "mp": "xxx",
        "mp___UNI__17E377B": "xxx"
    },
    "third_party": {
        "mp_weixin": {
            "session_key": "xxx"
        }
    },
    "register_env": {
        "appid": "xxx",
        "uni_platform": "mp-weixin",
        "os_name": "ios",
        "app_name": "uni-ai-chat",
        "app_version": "1.0.0",
        "app_version_code": "100",
        "channel": "1001",
        "client_ip": "127.0.0.1"
    },
    "register_date": 1697083704626,
    "dcloud_appid": [
        "xxx"
    ],
    "last_login_date": 1697083705053,
    "last_login_ip": "127.0.0.1",
    "token": ["xxx"]
}
```

假设我们要提供给用户可以修改用户名的操作，那么我们可以新建一张如下的数据库表：

| 字段名        | 字段说明                                         |
| ------------- | ------------------------------------------------ |
| `id`          | 默认字段，唯一，主键                             |
| `user_openid` | 用户的`openid`，用于识别用户身份，唯一，作为索引 |
| `user_name`   | 用户名，可供用户修改                             |

现在问题来了，我们用户登录的接口是`uni-id-co`封装好的，返回的有用信息只有`token`，那么我们在登陆时如何向这张表插入数据呢

我们可以使用该`token`向原有的`uni-id-users`表请求`token`对应的数据，然后提取其中的`openid`，然后写入这张表即可

然后我们可以给一个默认的用户名，直接使用`openid`都可以，然后我们可以有选择性地将一些用户数据存储在客户端的本地存储中





