> 本篇博客讲一下`Vue`使用动态添加路由时的路由守卫的注意事项，如有错误或更多建议，欢迎评论指出



### 动态路由

> Vue中的动态路由是由`addRoute()`方法实现的，既可以添加一级路由，也可以为不同模块添加子路由

```js
import router from '@/router/index.js'

router.addRoute({ 路由对象 }) // 添加一级路由
router.addRoute('一级路由名',{ 路由对象 }) // 添加子路由
```

---

### 路由守卫

* 这里主要讲全局路由守卫：

```js
router.beforeEach((to,from,next)=>{ // 这里的回调有时候还可能是async
  ... // 处理逻辑
  next()
})
```

* 有时候我们会在**进入首页之前**在路由守卫中请求用户的路由：  
  *注意：*
  1. 对于不同用户的权限，我们需要动态添加路由
  2. 在动态添加路由之后我们不能直接`next()`放行，而是需要重新走一遍`next(to.path)`，以保证路由表已经存在

```js
router.beforeEach(async (to,from,next)=>{ 

  if(进入到某个页面){

    ... // 请求到路由表

    router.addRoute('一级路由名',{ 路由表中每个路由对象 })
    next(to.path) // hack方法，确保路由存在，否则可能会出现白屏
  }

  next()
})
```