# React快速上手

想学`react`很久了，但是之前一下太多东西学不来，最近终于开始学习`react`，这篇笔记就作为自己的学习记录吧

主要是对`react`的知识点做一下总结，目的是快速上手

仅供自己作为学习总结和参考，讲的不太全也不太好，见谅



### 一、React项目结构

这里主要通过`react`的项目结构，来讲一下`react`的一些基础知识点





### 二、React Hooks

`react hooks`可以说是`react`的一大特点，函数的使用非常灵活，这里仅总结一些常用的`hook`



##### useState

这个`hook`就是用于构建响应式变量的，用法比较简单，只需要传一个参数，只有赋值作用

```react
import './App.css'
import React, { useState } from 'react'

function App() {
  const [val, setVal] = useState(0)

  return (
    <>
      <p>{ val }</p>
      <button onClick={ () => setVal(1) }>点击我修改值</button>
    </>
  )
}
export default App
```



##### useReducer

这是`useState`的升级版，可以处理更复杂的操作

使用`useReducer`时，它会向下传递一个`dispatch`函数

这个`dispatch`函数会一直持续存在，不会因为重新渲染而改变，相比`callback`，可以优化性能，这个暂时先了解一下即可



总结一下使用方法：

1. 定义一个`reducer`函数，用于处理传过来的旧`state`，返回新`state`
2. 使用`useReducer`时，用`state`来接收返回的新`state`的值
3. 调用`dispatch`会触发`reducer`函数，并会将这一轮的`state`值传过去

注意：数组中的`state`仅用于接收新的状态，它实际上是一个**对返回的值的引用**；`reducer`的`state`参数会把每一轮的值自动传进去

```react
import './App.css'
import React, { useReducer } from 'react'

function App() {
  function reducer(state, action) {
    console.log('传过来的参数：', action)
    return state + 1
  }

  // state用于接收返回的新state
  const [state, dispatch] = useReducer(reducer, 0)

  return (
    <>
      <p>{ state }</p>
      <button onClick={ () => dispatch('你好') }>点击我</button>
    </>
  )
}
export default App
```



##### useContext

这个其实跟`vue`里面的`provide/inject`比较像

就是父组件提供数据和方法，传给子组件，子组件调用父组件的方法，触发方法修改父组件的数据

然后子组件还可以将这些数据传给它的子组件，可以无限套娃，但是不建议



解读一下这段代码：

1. 我们在父组件中创建了一个`context`，这个作为上下文进行传递

2. 继续定义一个**需要传给子组件**的`state`
3. 利用`context`的`provider`，把`state`作为`value`传给子组件
4. 子组件用`props`接收这个`context`，然后绑定`context`中的方法

总的来说，就是：**父组件createContext，子组件useContext**

```react
import './App.css'
import React, { useContext, useState } from 'react'

function App() {
  // 在父组件中创建context
  const ValContext = React.createContext()
  // 在父组件中创建state
  const [val, setVal] = useState(1)

  const changeVal = () => {
    setVal(3)
  }

  return (
    // 把state作为context的value提供给子组件
    <ValContext.Provider value={ { val, changeVal } }>
      {/* 子组件同时需要接收props参数作为context */}
      <MyChild ValContext={ ValContext }></MyChild>
    </ValContext.Provider>
  )
}

function MyChild(props){
  const { val, changeVal } = useContext(props.ValContext)

  return(
    <>
      <button onClick={ changeVal }>点击修改值</button>
      <p>值为：{ val }</p>
    </>
  )
}

export default App

```

运行结果如下图所示，可以看出，子组件的`val`实际上是**对父组件的对应变量的引用**

![](http://ldmblog.ifoodin.com/20231007203039.png)



##### useRef

这个`hook`最主要的作用就是访问`dom`元素，就相当于`vue`里面给`dom`元素打一个`ref`标记

通过`ref`对象的`current`属性，我们可以获取被标记的`dom`元素

同时，通过`current`属性，也可以用来绑定一个可变的值，但是我感觉这个用的不多，暂时过一下就好了，主要还是它的标记的作用

```react
import React, { useRef } from 'react'

export default function FocusButton() {
  const inputEl = useRef(null)
  const onButtonClick = () => {
    // 一定要通过current属性来获取被绑定的dom
    inputEl.current.focus()
  };

  return (
    <>
      <input ref={ inputEl } type="text" />
      <button onClick={ onButtonClick }>文本框聚焦</button>
    </>
  )
}
```



##### useEffect

这是一个处理副作用的`hook`，它的回调函数会在依赖的变量改变时触发

要注意：当依赖项是对象等引用类型时，`react`比较的是它的**地址**有没有改变

还有，在组件挂载/重新加载的时候，这个`hook`的回调函数就会执行

```react
import './App.css'
import React, { useEffect, useState } from 'react'

function App() {
  const [var, setVar] = useState(0)
    
  useEffect(() => {
    alert('变量var改变了')
    // 处理其他副作用...
  }, [var])
    
  return (
    <>
      <p>{ state }</p>
      <button onClick={ () => setVar(1) }>点击我</button>
    </>
  )
}
export default App

```



##### useCallback

这个`hook`用于缓存一个回调函数本身，当组件重新渲染时，不会重新创建这个函数

也就是说，这个函数由头到尾都是同一个，只有当依赖项改变时，才会重新创建，可以优化性能

```react
import './App.css'
import React, { useCallback, useState } from 'react'

function App() {
  const [val, setVal] = useState(0)
    
  const cacheFunc = useCallback(() => {
    console.log('依赖项发生改变时会重新创建这个函数')
  }, [val])
    
  return (
    <>
      <p>{ val }</p>
      <button onClick={ () => setVal(1) }>点击我</button>
    </>
  )
}
export default App
```



##### useMemo

`useMemo`也是缓存作用，但它缓存的是一个值（变量），依赖项不变时，该变量不会重新创建

这个`hook`可以在给子组件传值时，用作优化

例如下列代码，如果不使用`useMemo`，那么子组件每次重新渲染时，`userInfo`变量都会重新创建一次

```react
const [count, setCount] = useState(0);

const userInfo = useMemo(() => {
  return {
    // ...
    name: "Jace",
    age: count
  }
}, [count])

return <UserCard userInfo={ userInfo }>
```





### 三、React Router

`react`路由，用于管理各个组件的渲染

因为东西有点多，所以仅作简单介绍，详情可参考这篇博客：[React路由最全使用(N) - 掘金 (juejin.cn)](https://juejin.cn/post/7229493617365712953#heading-37)



##### BrowserRouter组件

这个组件在整个`react`应用中，只会在`index.js`里面使用一次，用于包裹要进行路由管理的组件

这个和`blazor`的`Router`就非常相似，整个应用只配置一次

```react
import React from 'react'
import ReactDOM from 'react-dom/client'
import './index.css'
import App from './App'
import { BrowserRouter } from 'react-router-dom'

const root = ReactDOM.createRoot(document.getElementById('root'))
root.render(
  <BrowserRouter>
    <App></App>
  </BrowserRouter>
)
```



##### Route组件

`Route`组件就是是路由出口，需要路由的组件最终会匹配并渲染在这个位置

它接收`path`和`element`两个参数，分别表示路由的路径和要渲染的组件

```react
<Route path='/child' element={ <Child/> }></Route>
```



##### Routes组件

`Routes`组件用于包裹`Route`组件，来进行路由群组、分级的管理

```react
<Routes>
    <Route path='/' element={ <Home/> }></Route>
</Routes>
```



##### 嵌套路由

嵌套路由直接在`Route`里面嵌套就行了

```react
<Routes>
    <Route path='/' element={ <Home/> }>
        <Route path='/child' element={ <Child/> }></Route>
    </Route>
</Routes>
```

然后我们在需要渲染子组件的父组件中通过`Outlet`组件来指定子组件渲染的位置

```react
import React, { useCallback, useState } from 'react'
import { Outlet, Route, Routes } from 'react-router-dom'

import Child from './Child'

function Home() {
  return (
    <>
      <p>Home页面</p>
      {/* 指定子组件的渲染位置 */}
      <Outlet/>
    </>
  )
}
export default Home
```





### 四、状态管理库Mobx

因为学长推荐用`mobx`，所以就先学习`mobx`，其核心就是`store`、`state`、`action`、`computed`

这里仅作简单使用的介绍，详情请参考以下两篇博客：

[MobX 上手指南 - 掘金 (juejin.cn)](https://juejin.cn/post/6921544116094533646)

[react.js - React MobX 开始 - GoCoding - SegmentFault 思否](https://segmentfault.com/a/1190000041193632#item-3)



##### 对象实例store

我们先来看最简单的`store`的使用，我们直接定义一个对象

这里`makeAutoObservable`是`mobx`创建一个`store`最基础的`API`

这里我们直接把这一段写在一个`.js`文件中，然后`export`这个`store`，就可以在各个组件中使用了

```react
const store = makeAutoObservable({
    count: 1,
    get doubleCount() {
        return this.count*2
    },
    increse() {
        this.count++
    },
    decrease() {
        this.count--
    }
})

export default store
```



##### 模块化store

当我们有多个`store`的时候，我们就可以创建一个根`store`来进行管理，然后每个业务`store`就是这个根`store`的子模块

为了方便管理，我们需要创建`class`来作为`store`模型

然后，为了方便在组件间传递，我们用`useContext`机制来进行封装

```react
// counter.Store.js

import { makeAutoObservable } from 'mobx'

class Counter {
    count = 1
	
	constructor() {
        makeAutoObservable(this)
    }

	increase() {
        this.count++
    }

	decrease() {
        this.count--
    }
}

const counter = new Counter()
export default counter
```

```react
// root.Store.js

import { createContext, useContext } from 'react'
import { makeAutoObservable } from 'mobx'
import counter from './counter.Store.js'

class RootStore {
    constuctor() {
        // 给rootStore的属性赋值，作为子模块
        this.counter = counter
        {/* 以后想要添加什么另外的子模块都可以直接在这里加... */}
    }
}

// 包装成context
const rootStoreContext = createContext(new RootStore())
const useStore = () => React.useContext(rootStoreContext)
export default useStore // 暴露一个函数，组件要使用store的，调用这个函数即可，可以直接获取根store实例
```



##### 在组件中使用store

在`react`组件中，我们直接`import`定义好的`store`即可，这里以我们上面的对象实例`store`为例

这里注意，想要让组件的视图随`state`发生变化，就要用`mobx`提供的`observer`来包装组件，否则`state`对于组件来说就不是响应式的

```react
import counter from './counter.js'
import { obeserver } from 'mobx'

function Test() {
    return (
    	<div>{ counter.count }</div>
        <button onClick={ counter.increase }></button>
    )
}

// 响应式组件包装
export default observer(Test)
```

