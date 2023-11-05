# 使用TS开发Vue项目

第一次看`ts`时，给我的印象就是除了写法比较抽象之外，还有项目中的配置项非常杂乱，老是容易忘

最近摆了两个星期，想要找点事做，打算就从`ts`开始，先整理一下，顺便纠正一下自己之前错误的理解

那些什么基本类型的使用我就不提了，我就只整理我认为值得整理的地方





### tsconfig.json配置文件

这是`TypeScript`编译器必需的，如果没有`tsconfig.json`文件，编译器将不会编译`TypeScript`文件



##### references配置项

`tsconfig.json`文件是`ts`项目最主要的配置文件，在`vue3`项目中并没有在`tsconfig.json`中直接进行配置

而是使用了`references`的配置项来引用了`tsconfig.app.json`和`tsconfig.node.json`的配置文件，来合成一个总的配置文件

```json
{
  "files": [],
  "references": [
    {
      "path": "./tsconfig.node.json"
    },
    {
      "path": "./tsconfig.app.json"
    }
  ]
}
```



##### include配置项

`include`配置项用于指定需要编译的`ts`文件，我们先来看看`tsconfig.app.json`中的`include`配置项

```json
"include": [
    "env.d.ts", 
    "src/**/*", 
    "src/**/*.vue"
],
```

这里就将这三种包含的`ts`代码的文件进行编译，只有编译为`js`后，项目才能正常运行，一般项目中的这个配置我们无需改动





### .d.ts声明文件

声明文件应该是项目中应用`ts`时，最让人头疼的地方，因为引入`module`时经常报错就是因为`.d.ts`文件没有找到

那么`ts`项目是如何找到并编译`.d.ts`文件的呢，我也一直很迷糊，所以现在要好好分析一下



##### include配置项

首先，我们**自定义**的`.d.ts`声明文件一般在`include`配置项下引入

但是实际上我们在自定义的`.d.ts`文件中声明的`type`，一般引入时都是`import`，所以在这里`include`也没有用处

所以这个`include`主要是针对在`.d.ts`中使用`declare`的

只要`include`了`.d.ts`文件，那么就可以直接使用`declare`的东西而不用`import`

```json
"include": [
    "env.d.ts", 
    "src/**/*", 
    "src/**/*.d.ts", // 引入声明文件
    "src/**/*.vue"
],
```

然后，我们一般在`src`文件夹下新建一个`types`文件夹，专门用于存放`.d.ts`声明文件，这样我们导入类型就可以直接使用了



##### compileOptions下的types配置项

我们引入的第三方库的类型声明包一般都在`compileOptions`下的`types`中配置

```json
"types": [
    "@dcloud/types",
    "@uni-helper/uni-ui-types"
]
```

如果我们不设置`typeRoots`项，那么它就会先在`uni_modules/@types`文件夹下查找，然后再查找`uni_modules`文件夹下的

如果我们设置了`typeRoots`项，那么它不会查找上面两项，而是**只查找**在`typeRoots`项中配置的路径，就是说**会直接覆盖**默认路径

例如，我们做了如下的配置，那么它就会以`./test`为根目录，结合`types`项，查找第三方库的`.d.ts`声明文件，不会再查找其他路径

```json
"compileOptions": {
    "typeRoots": ["./test"]
}
```

所以我们如果要自定义`typeRoots`，那么往往会加上`"./node_modules/@types"`的路径，以方便查找第三方库的类型声明文件

```json
"compileOptions": {
    "typeRoots": [
        "./test",
        "./node_modules/@types",
    ]
}
```





### declare声明的使用

在`.d.ts`文件中，顶级语句除了`export`，还可以用`declare`

`declare`可以声明变量、`type`、`interface`、函数、`module`、命名空间等等

`declare`与`export`不同，就是它在`tsconfig.json`中的`include`中被包括了的话，那么就可以不用`import`即可使用



##### declare module

这个用法，顾名思义就是声明一个模块

在一个`declare`模块字面量里面可以使用`export`，但是不用也可以，建议不用，不然容易搞混

一般声明模块都是对第三方的包的模块进行扩展，我们举个例子看看，这样就可以扩展类的属性

```typescript
declare module 'vue' {
    interface Component {
        name: string
    }
}
```



