> 复习一下vue的基础知识，这篇博客讲一下v-model



### v-model本质

* 在一般的表单项中，v-model用于双向数据绑定，其本质是一个语法糖，可以拆成`:value`和`@input`两部分

```js
<input type="text" v-model="content">

// 相当于：
<input type="text" :value="content" @input="content = this.$event.target.value">
```

---

### 对自定义组件使用v-model

* 将v-model拆解后，可以认为其就是传入了一个默认为`value`的`prop`，并且设置了一个默认为`input`自定义事件，来进行**组件间通信**

```js
// 父组件
<template>
  <SonComponent v-moedl="content"></SonComponent>
</template>

// 相当于：
<template>
  // arguments是子组件传回来的参数
  <SonComponent :value="content" @input="content = arguments[0]"></SonComponent>
</template>
```

```js
// 子组件
<template>
  <div>
    <input :value="value" type="text">
    <button @click="handleClick"></button>
  </div>
</template>

<scrpit>
export default {
  props: {
    value: {
      type: Number,
      default: 1, 
    }
  },
  methods: {
    handleClick() {
      // args是参数，一般这样的只传回去一个参数 
      this.$emit('input',args)
    }
  }
}
</scrpit>
```

* 如果觉得子组件中的`props`的名字和别的`props`冲突了，vue还提供了自定义`model`

```js
<template>
  <div>
    <!--使用修改后的props名--> 
    <input :value="myValue" type="text">
    <button @click="handleClick"></button>
  </div>
</template>

<scrpit>
export default {
  model: {
    prop: 'myValue', // 默认为value
    event: 'myEvent', // 默认为input
  },
  props: {
    // 使用修改后的prop名
    myValue: {
      type: Number,
      default: 1, 
    }
  },
  methods: {
    handleClick() {
      // 使用修改后的事件名
      this.$emit('myEvent',args)
    }
  }
}
</scrpit>
```