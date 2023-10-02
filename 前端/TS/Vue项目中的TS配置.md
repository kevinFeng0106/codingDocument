# Vue项目中的TS配置

学习了`ts`和一些`vue`的配置之后，老是会忘记

这篇文档就来整理巩固一下`ts`及其相关配置



### 一、路径别名及映射

> 在项目中，经常会将`@`符作为`src`文件夹的映射及别名，虽然这不是`ts`才有的配置，但是就放在这里一起说吧，我们来看一下怎样配置路径解析



##### 1. 第一种方式

我们可以在`vite.config.ts`文件中添加如下的`alias`配置：

```typescript
export default defineConfig({
    // 省略其他配置...
    resolve: {
        alias: {
            "@": resolve(__dirname, "./src")
        }
    }
})
```



##### 2. 第二种方式

我们还可以在`tsconfig.json`文件中作如下配置：

```typescript
{
    "compileOptions": {
        // 省略其他配置...
        "baseUrl": ".",
        "paths": {
            "@/*":["src/*"]
        }
    }
}
```





### 二、include选项

> 在`tsconfig.json`文件的`include`选项中，被包含的文件会被`ts`自动编译，也就是相当于`vue`会自动解析执行这些文件，就不用我们手动引入再去执行了



举个例子：

这里就是所有带`.ts`、`.d.ts`、`.tsx`、`.vue`后缀的文件都会被自动编译

我们在引入`Element Plus`组件库时，可以利用这个`include`来做一些自动导入

```json
"include": [
    "src/**/*.ts",
    "src/**/*.d.ts",
    "src/**/*.tsx",
    "src/**/*.vue",
]
```





### 三、types选项

> 在`tsconfig.json`中还有一个`types`选项，这是用来扩展`ts`的类型的，例如我们引入了一些第三方库，需要对其中的一些参数作解析，就可以引入这个库的`types`文件



例如我们引入了`Element Plus`组件库

我们就可以一同引入它的`types`声明文件：

```json
"types": [
    "element-plus/global"
]
```





### 四、declare module语句

> 通常我们会在一些被自动编译的文件类型中使用`declare module`来进行类型扩展



##### 1. 自定义模块

声明`custom`模块，名字可以随便起：

```typescript
declare module 'custom' {
    // 定义某个类型
    const myComp: typeof ...
}
```

使用时，引入模块即可：

```typescript
import * as custom from 'custom'

// 使用模块中声明的类型
const func = (comp: custom.myComp) => {
    // ...
}
```



##### 2. global模块

使用`declare global`的话，就不用显式地导入模块再使用了：

```typescript
declare global {
    // myComp无需再导入模块，直接使用即可
    const myComp: typeof ...
}
```





### 五、InstanceType获取组件实例类型

> `InstanceType`可以获取组件实例类型，在我们使用组件实例的方法和属性时，有不错的辅助作用，使用`InstanceType`时，需要传入一个类型作为参数，一般我们先引入组件，然后配合`typeof`来获取组件类型，然后将这个类型传入`InstanceType`中



举个例子：

```typescript
// 引入组件
import MySwiper from '...path'

// 声明组建的实例类型
export type MySwiperInstance = InstanceType<typeof MySwiper>
```



### 六、Component声明组件类型

> 在小程序中，对于自定义或插件市场的组件，可以通过`Component<T>`来对该组件的类型进行包装，成为一个组件类型



