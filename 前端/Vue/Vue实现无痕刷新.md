# Vue实现无痕刷新

首先我们要弄清楚`vue`的无痕刷新依赖的是组件的`onMounted`生命周期

当组件重新挂载时重新请求数据，以达到刷新的效果，那么我们就要想办法--如何让组件重新挂载

我们先来尝试一下几种方法，然后再判断这些方法可不可行



### 一、在组件内部加上v-if

我们在`template`中加上`v-if`来尝试让**组件内部**的元素重新渲染

尝试过后，可以发现组件内部的元素确实重新渲染了，但是`onMounted`生命周期并没有触发

这是因为我们是在组件内部进行操作，并没有让整个组件重新挂载，所以这个方法并不可行

```vue
<script setup>
import { ref, onMounted} from 'vue'
import  { getListAPI, postListDataAPI } from './request/api'

const isShow = ref(true)

// 添加的数据
const addText = ref('')

// 列表数据
const listData = ref([])

// 获取列表数据
const getList = async () => {
  const res = await getListAPI()
  console.log('列表数据', res) 
  listData.value = res
}

// 向列表添加数据
const addData = async () => {
  isShow.value = false
  const res = await postListDataAPI({ name: addText.value })
  isShow.value = true
  console.log('添加数据', res)
}

onMounted(() => {
  console.log('组件挂载了')
  getList()
})
</script>

<template>
  <template v-if="isShow">
    <label for="">
      <input type="text" class="add-data" v-model="addText">
    <button @click="addData">添加</button>
    </label>
    <ul class="test-list">
      <li class="lsit-item" v-for="item in listData" :key="item.id">{{ item.name }}</li>
    </ul>
    </template>
</template>
```





### 二、在顶级router-view组件中设置v-if

既然我们想让整个组件重新挂载，那么就应该在它的父组件中设置条件渲染，这里我们先使用`router-view`组件试一试

试过之后发现是可行的，虽然还是有点小瑕疵，但可以避免出现白屏



在`App.vue`中，我们使用`router-view`，并且`provide`出去一个全局刷新方法

```vue
<script setup>
import { nextTick, provide, ref } from 'vue'

// 组件是否显示
const isRouterShow = ref(true)

// 刷新方法
const refreshRouter = async () => {
  isRouterShow.value = false
  // 等渲染完dom之后再进行下一步操作
  await nextTick()
  isRouterShow.value = true
}

// 将刷新方法provide出去
provide("refreshRouter", refreshRouter)
</script>

<template>
  <router-view v-if="isRouterShow"></router-view>
</template>
```



然后我们把之前写的封装成一个`list.vue`组件，然后`inject`全局刷新方法

```vue
<script setup>
import { ref, onMounted, inject} from 'vue'
import  { getListAPI, postListDataAPI } from '../../request/api'

// 注入全局刷新方法
const refreshRouter = inject("refreshRouter")

// 添加的数据
const addText = ref('')

// 列表数据
const listData = ref([])

// 获取列表数据
const getList = async () => {
  const res = await getListAPI()
  console.log('列表数据', res) 
  listData.value = res
}

// 向列表添加数据
const addData = async () => {
  const res = await postListDataAPI({ name: addText.value })
  refreshRouter()
  console.log('添加数据', res)
}

onMounted(() => {
  console.log('组件挂载了')
  getList()
})
</script>

<template>
    <label for="">
        <input type="text" class="add-data" v-model="addText">
    <button @click="addData">添加</button>
    </label>
    <ul class="test-list">
        <li class="lsit-item" v-for="item in listData" :key="item.id">{{ item.name }}</li>
    </ul>
</template>
```

