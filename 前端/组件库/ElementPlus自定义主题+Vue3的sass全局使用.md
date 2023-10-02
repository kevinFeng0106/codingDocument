> 首先在这里，要爆骂element的文档，怎么写的不清不楚的，真的是一坨
> 然后这篇博客主要讲一下ElementPlus中如何配置自定义主题



### 安装sass

* `ElementPlus`可以用`sass`来修改主题

```bash
npm install sass --save
```

---

### 配置自定义主题文件

* 在`src`下的`styles`文件夹中新建`element`文件夹，再在`element`中新建`index.scss`文件

```scss
/* 只需要重写你需要的即可 */
@forward 'element-plus/theme-chalk/src/common/var.scss' with (
  $colors: (
    'primary': (
      'base': green,
    ),
  ),
)
```

---

### 修改vite.config.js

* element的文档中缺了`{ importStyle: 'sass' }`这一步，样式一直不生效，折磨了我大半个晚上，我tm还以为哪里出问题了，结果官方文档半点没提这个，真是服了

```js
import { fileURLToPath, URL } from 'node:url'
import AutoImport from 'unplugin-auto-import/vite'
import Components from 'unplugin-vue-components/vite'
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'

import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [
    vue(),
    AutoImport({
      resolvers: [
        ElementPlusResolver({ importStyle: 'sass' }),
      ],
    }),
    Components({
      resolvers: [
        // 官方文档这里少了
        ElementPlusResolver({ importStyle: 'sass' }),
      ],
    }),
  ],
  css: {
    preprocessorOptions: {
      scss: {
        // 这里实际上是将样式文件中生声明的变量全局引入
        // 这也是项目中scss变量的规范引入方式，只是element自定义主题的时候用到了这种引入方法，我们自己也可以全局引入想要的变量
        additionalData: `@use "@/styles/element/index.scss" as *;`,
      },
    },
  },
  resolve: {
    // 实际的路径转化
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url))
    }
  }
})
```

* 最后总结一下，`element`自定义主题用到了`scss`的全局变量引入，这不是`element`自定义主题所特有的，我们自己也可以用这种方式
* 然后，再抱怨一下`element`的官方文档，真的亏贼