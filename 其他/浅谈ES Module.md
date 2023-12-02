# 浅谈ES Module

最近在写一个`react`项目的时候遇到了一个模块引入的问题，所以决定写一篇博客来总结`es module`的使用及注意事项



### 一、问题描述

大家应该都知道，在`react`中，使用组件有特定的**标签**格式

通常在使用`import ... from ...`引入组件后，可以直接拿这些“对象”，来当作组件使用

但是如果我想在组件`function` 中导入再使用呢？这里我会说明不同的两种导入方式的详细使用





### 二、顶级声明导入

顶级声明的导入方式就是`import ... from ...`了，这种方式必须写在`js` 文件的第一级

该方式的导出有默认导出：`export default`和命名导出：`export { ... }` 两种

如果我们想用默认导入的方式引入命名导出是不行的，但是我们可以使用以下方式：

```js
import * as moduleName from '...path'
```

这样我们就可以通过这个`moduleName`对象，使用其中的字段，来使用命名导出的模块了





### 三、组件function内导入

想要在组件`function`内导入，我们就要使用`import(...)`函数

该函数返回的模块会被包装成一个`Promise`，所以我们接收时，要用`async await`或者`then`

这就意味着这样子引入必然会是异步的操作，这个时候我们就要注意了，你在组件`function`内使用异步是要非常谨慎的

你必须要让`react`检测到你所做的改变，例如：

我们在组件初始化时立即执行了一个异步函数，该异步函数里面的代码又是表现为同步的（并非完全异步）

当触发了`setAntdIcons`之后，`react`就检测到了`state`的变化，它就会重新渲染了

```js
function myComponent() {
    const [antdIcons, setAntdIcons] = useState(null)
    useEffect(() => {
		(async () => {
            const icons = await import('@ant-design/icons')
            setAntdIcons(icons)
        })()
    }, [])
}
```

