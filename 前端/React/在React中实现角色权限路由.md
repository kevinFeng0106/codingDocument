# 在React中实现角色权限路由

我在找了很多方法，几乎全部都是没有动态地改变初始路由的，都是只配置了一次路由，也就是只做了路由跳转的校验

而这篇博客就来讲讲如何在`react`中配合后端根据**角色权限**动态地**改变路由表**并渲染路由





### 一、整体思路

我们使用`useRoutes`来动态创建路由，当用户点击登录成功后，请求后端得到权限路由表

然后改变`App.jsx`中的响应式监测数据，触发`App.jsx`组件的重新渲染，重新创建路由



##### 1. 路由表创建

首先我们给一个默认路由表，就是用户登录之前的路由表，只有登录（或者加上注册）页面

直接写在`App.jsx`中不太好，我们用`mobx`来进行路由表的管理

```jsx
class RouterStore {
  newRoutes = [
    {
      path: '/',
      element: <Home/>
    },
  ]
  constructor() {
    makeAutoObservable(this)
  }
  setNewRoutes = (data) => {
    runInAction(() => {
      this.newRoutes = data
    })
    console.log('当前mobx路由数组', this.newRoutes)
  }
}

// 这里实际上应该用createContext包裹一下
const routerStore = new RouterStore()
const useRouterStore = () => routerStore
export { useRouterStore }
```



##### 2. 在App.jsx中监测mobx的数据变化

注意一下这里的`useState`和`useEffect`，我们实际上是用`state`中的路由表来`useRoutes`的

`mobx`中的`newRoutes`路由表只是用于承接后端的数据，用于被监听然后触发`App.jsx`组件重新渲染

而`useEffect`中要注意是否会触发**组件渲染的重复死循环**

```jsx
import './App.css'
import { useNavigate, useRoutes } from 'react-router-dom'
import Home from './pages/home/index.jsx'
import { useEffect, useRef, useState } from 'react'
import { useRouterStore } from './store/routes.Store.js'
import Login from './pages/login/index.jsx'
import { observer } from 'mobx-react-lite'


function App() {
  const routerStore = useRouterStore()

  const navigate = useNavigate()

  const [routes, setRoutes] = useState([
    {
      path: '/',
      element: <Home/>
    },
  ])

  useEffect(() => {
    setRoutes(routerStore.newRoutes) // 更新路由state，触发组件重新渲染
    console.log(111)
  }, [routerStore.newRoutes]) // 监听mobx的路由表的变化

  const element = useRoutes(routes)

  return (    
    <div className="App">
      <button onClick={ async () => { // 异步更新
        await routerStore.setNewRoutes([
          {
            path: '/',
            element: <Home/>
          },
          {
            path: '/login',
            element: <Login/>
          },
        ])
        navigate('/login?name=ldm')
      } }>点击修改</button>
      { element }
    </div>
  );
}

export default observer(App) // mobx响应式组件
```



##### 3. mobx路由表持久化

虽然这样子我们点击之后好像可以添加路由并跳转了了，但是我们点击刷新后试试，发现路由表又没了

这是因为刷新后`mobx`的数据都恢复初始状态了，就只有`Login`一个路由了



##### 4. mobx持久化工具基本使用

我们使用`mobx-persist-store`这个包来对`mobx`进行持久化，输入如下命令安装

```shell
npm install --save mobx-persist-store
```

然后开启持久化，基本的使用如下，详情请参考官方文档：[mobx-persist-store - npm (npmjs.com)](https://www.npmjs.com/package/mobx-persist-store)

```js
class RouterStore {
  newRoutes = [
    {
      path: '/',
      element: <Home/>
    },
  ]
  constructor() {
    makeAutoObservable(this)
    makePersistable(this, { // 开启持久化
      name: 'learn-router-store', // 指定存储的key
      properties: ['newRoutes'], // 指定要持久化的字段
      storage: window.localStorage, // 指定存储方式
      
    })
  }
  setNewRoutes = (data) => {
    runInAction(() => {
      this.newRoutes = data
    })
    console.log('当前mobx路由数组', this.newRoutes)
  }
}
```



##### 5. 修改持久化配置

首先我们在`jsx`文件中的一个组件在控制台输出后，可以看到是一个对象

所以按照我们上面的存储的写法，我们存进本地存储和从本地存储取出来的都是一个组件对象

然后我们直接把一个`object`作为`element`内容，这在`react`中是不允许的，它会报如下的错误：

```shell
Error: Objects are not valid as a React child (found: object with keys {list}).
```

所以我们应该定制一下序列化器和反序列化器，并且使用懒加载来实现赋值一个组件而不是一个对象：

```jsx
import { makeAutoObservable, runInAction } from "mobx"
import Home from "../pages/home"
import Login from "../pages/login"
import { makePersistable } from "mobx-persist-store"
import { Suspense, lazy } from "react"

// 动态引入
const genComponent = (path) => {
  console.log('动态引入组件的路径', `../pages${path}`)
  const Module = lazy(() => import(`../pages${path}`))
  return <Module/>
}

class RouterStore {
  newRoutes = [
    {
      path: '/',
      element: <Home/>
    },
  ]
  constructor() {
    makeAutoObservable(this)
    makePersistable(this, {
      name: 'learn-router-store',
      properties: [
        {
          key: 'newRoutes',
          serialize: (value) => {
            console.log('存到本地之前的值', value)
            console.log('存到本地之后的值', JSON.stringify(value.map(item => item.path)))
            return JSON.stringify(value.map(item => {
              return { path: item.path }
            }))
          },
          deserialize: (value) => {
            console.log('从本地存储拿到的路由表转换为对象前的值', value)
    
            return JSON.parse(value).map(item => {
              if(item.path === '/') {
                return {
                  path: '/',
                  element: genComponent('/home')
                }
              } 
              else {
                return {
                  path: '/login',
                  element: genComponent(item.path)
                }
              }   
            })
          }
        },
      ],
      storage: window.localStorage,
      
    })
  }
  setNewRoutes = (data) => {
    runInAction(() => {
      this.newRoutes = data
    })
    console.log('当前mobx路由数组', this.newRoutes)
  }
}
```



##### 6. 懒加载报错问题

像上面这样子写好了之后，可能会报如下的错误：

```shell
A component suspended while responding to synchronous input. This will cause the UI to be replaced with a loading indicator. To fix, updates that suspend should be wrapped with startTransition
```

这是因为我们在`react`组件懒加载的时候，发生页面跳转，但是可能由于 网络的原因，路由跳转了，但是组件的内容还没有跳转过去，所以就报错了；我们只要在懒加载的组件外层包上一个`<Suspense>`，然后给一个`fallback`就行了：

```jsx
import { Suspense, lazy } from "react"

// 动态引入
const genComponent = (path) => {
  console.log('动态引入组件的路径', `../pages${path}`)
  const Module = lazy(() => import(`../pages${path}`))
  return (
    <Suspense fallback={ <div>稍等</div> }>
      <Module/>
    </Suspense>
  )
}
```

