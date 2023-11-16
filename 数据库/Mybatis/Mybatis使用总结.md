# Mybatis使用总结

`mybatis`是一个功能非常强大的数据库持久层框架

我之前在使用时都只是用到了最最基础的部分，所以这篇博客就来进一步了解一下其操作

`mybatis`分为注解操作和`mapper`文件映射操作，由于注解写法比较简单，这里就只总结`mapper`映射的写法

本篇博客会持续更新...





### Mapper映射文件

首先，`mapper`映射文件夹是对应了我们自己定义的`mapper`层的文件夹的

也就是说，我们需要在`resources`目录下新建一个与`java`文件夹下的`mapper`文件夹的目录结构**完全一样**的`mapper`目录

然后我们还需要在`resources`下的`mapper`文件夹内新建`mapper`接口对应的`xml`文件

注意：这里的`xml`文件名需要与我们`java`文件夹中的`mapper`接口的文件名**一模一样**

这里的`resources`就是我们常说的`classpath`

![](http://ldmblog.ifoodin.com/20231110082422.png)





### Mybatis配置

在使用`mybatis`时，，它是有默认的字段自动映射的，但是它是必须要数据库字段与实体属性类命名完全一致才可以

而我们在数据库中的字段一般是下划线分隔，在实体类中一般是驼峰命名法，这就需要我们手动开启下划线和驼峰命名法的自动转换

另外，在使用`mybatis`的`mapper`映射文件时，我们会设置到`resultType`和`paramType`等等

这是需要匹配实体类的完全限制路径的（也就是那一堆用“.”来连接的包前缀），而如果我们直接指定好实体类的包名，就可以只写类名了

在`yml`配置文件中书写如下配置

```yaml
mybatis:
  configuration:
    map-underscore-to-camel-case: true  # 开启驼峰命名和下划线命名的字段自动转换
  type-aliases-package: com.lordmoon.pojo # 实体类包路径前缀
```





