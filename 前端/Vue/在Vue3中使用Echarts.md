> 由于Vue是响应式的，而通常echarts的配置是一次性的，而且直接使用echarts还需要我们操作dom元素，这篇博客就讲一下一个在vue中使用的echarts库：[vue-echarts](https://github.com/ecomfe/vue-echarts/blob/main/README.zh-Hans.md)



### 库的下载

* 使用`npm`即可，注意因为是基于`echarts`的，所以要把`echarts`一起下了

```bash
npm install echarts vue-echarts
```

---

### 注意

* 如果要使用`echarts`**全局对象**的话，我们还需要先`cdn`引入`echarts`
* 将这一段放入`index.html`中：
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/echarts/5.4.3/echarts.min.js"
    integrity="sha512-EmNxF3E6bM0Xg1zvmkeYD3HDBeGxtsG92IxFt1myNZhXdCav9MzvuH/zNMBU1DmIPN6njrhX1VTbqdJxQ2wHDg=="
    crossorigin="anonymous" referrerpolicy="no-referrer"></script>
```

---

### 库的基本使用

* 先来看一个vue3中使用的demo
* 具体的解释详见代码注释
```js
<template>
  <v-chart class="chart" :option="option" />
</template>

<script setup>
// 需要引入很多东西
// 在用的时候，如果不知道要引入什么组件，可以先切换到控制台查看报错，它会告诉你如何引入的
import { use } from "echarts/core";
import { CanvasRenderer } from "echarts/renderers";
import { PieChart } from "echarts/charts";
import {
  TitleComponent,
  TooltipComponent,
  LegendComponent
} from "echarts/components";
import VChart, { THEME_KEY } from "vue-echarts";
import { ref, provide } from "vue";

use([
  CanvasRenderer,
  PieChart,
  TitleComponent,
  TooltipComponent,
  LegendComponent
]);

provide(THEME_KEY, "dark");

// 这里的ref里面的对象，我们可以用echarts官网中的配置直接替换
const option = ref({
  title: {
    text: "Traffic Sources",
    left: "center"
  },
  tooltip: {
    trigger: "item",
    formatter: "{a} <br/>{b} : {c} ({d}%)"
  },
  legend: {
    orient: "vertical",
    left: "left",
    data: ["Direct", "Email", "Ad Networks", "Video Ads", "Search Engines"]
  },
  series: [
    {
      name: "Traffic Sources",
      type: "pie",
      radius: "55%",
      center: ["50%", "60%"],
      data: [
        { value: 335, name: "Direct" },
        { value: 310, name: "Email" },
        { value: 234, name: "Ad Networks" },
        { value: 135, name: "Video Ads" },
        { value: 1548, name: "Search Engines" }
      ],
      emphasis: {
        itemStyle: {
          shadowBlur: 10,
          shadowOffsetX: 0,
          shadowColor: "rgba(0, 0, 0, 0.5)"
        }
      }
    }
  ]
});
</script>

<style scoped>
.chart {
  height: 400px;
}
</style>
```

---

### 最后

* 上面这种注册方式很臃肿，所以我们可以在`plugins`文件夹下的`echarts.js`中注册，然后在`main.js`中引入即可
* 但是注意，不是全部的引入都要放到`echarts.js`中，例如`provide`和`THEME_KEY`，还有`VChart`