> 在vue3中，对于数组，既可以用`ref`，也可以用`reactive`，但是好像更多人用`ref`,那么这两个有什么区别和需要注意的地方呢，这篇博客就通过实操来说明一下



### reactive的“失效”问题

* 如果我们先声明了一个`reactive`数组，并且用`v-for`绑定到模板上，那么在请求数据后**重新赋值**（重新用`reactive`包裹请求得到的数据，然后赋值给原来的数组），模板中并没有动态地更新数据，看起来好像`reactive`失效了

* 但是我们打印重新赋值后的数组会发现，它仍是响应式的

* 出现这种情况是因为我们一开始绑定到模板上的`reactive`数组是一个**引用**，重新赋值后，数组的引用改变了，但模板上**绑定的仍然是原来的引用**，所以是无法动态更新的

```js
<template>
  <div class="home">
    // 在模板上绑定数组
    <div v-for="item in arr" :key="item.id">{{ item.task }}</div>
  </div>
</template>

<script setup>
import { onMounted, ref, reactive, isReactive } from 'vue'
import { GetListAPI } from '@/apis/request.js'

let arr = reactive([{ task: '111' }])

// 请求数据列表
const getList = async () => {
  const res = await GetListAPI()
  console.log(res)
  // 重新用reactive赋值数组
  // 上面的模板数据却不更新了
  arr = reactive(res)
  // 打印发现仍然是响应式的
  // 但是引用已经改变
  console.log(isReactive(arr)) // true
}

onMounted(() => {
  // 挂载时请求数据
  getList()
})
</script>
```

---

### ref的好处

* ref在模板中使用时,会自动解包
* 在赋值时,使用`.value`赋值,**不会改变原有的引用**

```js
<template>
  <div class="home">
    <div v-for="item in arr" :key="item.id">{{ item.task }}</div>
  </div>
</template>

<script setup>
import { onMounted, ref, reactive, isReactive } from 'vue'
import { GetListAPI } from '@/apis/request.js'

// 使用ref
let arr = ref([])

const getList = async () => {
  const res = await GetListAPI()
  console.log(res)
  // 使用.value赋值,不改变原有引用
  // 此时模板中的数据会动态更新
  arr.value = res
  // 响应式也仍然保留
  console.log(isReactive(arr))
}

onMounted(() => {
  getList()
})
</script>
```