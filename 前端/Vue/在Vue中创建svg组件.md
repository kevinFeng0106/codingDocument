> 这篇博客讲一讲为什么即如何封装一个Vue的svg组件



> 先来看看svg的代码样例：

```js
<svg xmlns="http://www.w3.org/2000/svg" 
  xmlns:xlink="http://www.w3.org/1999/xlink" 
  style="position: absolute; width: 0; height: 0" id="__SVG_SPRITE_NODE__">
  <symbol xmlns="http://www.w3.org/2000/svg" 
    xmlns:xlink="http://www.w3.org/1999/xlink" 
    class="icon" viewBox="0 0 1024 1024" id="one">
    <defs><style type="text/css"></style></defs>
    <rect width="100%" height="100%" style="fill:rgb(153, 238, 172);stroke-width:2;stroke:rgb(0,0,0)">
    </rect>    
  </symbol>
</svg>

```

*是不是感觉非常臃肿，如果这样子直接贴到项目中，肯定是不行的*



> vue中自带了以下格式的svg使用方法：

```js
<svg><use :xlink:href="svg文件路径"/></svg>
```

*这样子使用起来就很方便清晰*



> 但是，还有没有更nb的方法呢？肯定是有的，我们可以封装一个vue的**svg组件**

1. 首先需要安装`svg-sprite-loader`

```bash
npm install svg-sprite-loader
```



2. 首先我们来看一下封装svg组件的结构：

![](http://ldmblog.ifoodin.com/20230707002445.png)

* 在`assest`目录下新建`icons`文件夹，然后在`icons`下新建一个`index.js`文件和一个`svg`文件夹
* `svg`文件夹存放`.svg`图片和我们封装的`SvgIcon.vue`组件



3. 再来看一下代码

* `index.js`
```js            
import Vue from 'vue'
import SvgIcon from './svg/SvgIcon.vue' // 引入封装的组件，路径其实可以随意，引入对了即可

Vue.component('svg-icon', SvgIcon) //全局注册组件，标签名为svg-icon

const req = require.context('@/assets/icons/svg', false, /\.svg$/) //请求资源的路径
const requireAll = requireContext => {
  // requireContext.keys() 数据：['./404.svg', './agency.svg', './det.svg', './user.svg']
  requireContext.keys().map(requireContext)
}
requireAll(req)
```

* `SvgIcon.vue`

```vue
<template>
  <i v-if="iconFileName.indexOf('el-icon-') === 0" :class="iconFileName" /> //可以使用element的样式
  <svg v-else class="svg-icon" aria-hidden="true" v-on="$listeners"> //对vue的svg进一步封装
    <use :xlink:href="`#icon-${iconFileName}`" rel="external nofollow" />
  </svg>
</template>

<script>
export default {
  name: 'SvgIcon',
  // props，方便传入标签的名字
  props: {
    iconFileName: {
      type: String,
      required: true
    }
  }
}
</script>

<style scoped>
// 一些样式
.svg-icon {
  width: 1em;
  height: 1em;
  overflow: hidden;
  vertical-align: -0.15em;
  fill: currentColor;
}
</style>
```



4. 最后，在`main.js`引入我们在`icons`下的`index.js`

```js
import '@/assets/icons/index.js'
```



5. 具体使用

```js
<svg-icon icon-file-name="svg文件名" style="margin:0 7px" />
```

*封装好组件后，还可以配合后端实现动态渲染svg图标，非常的好用*