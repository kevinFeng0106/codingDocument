> 看了挺多教程之后发现Vue创建项目的方式很多，现在就来整理一下我常用的两个Vue项目创建方法



### Vue2项目创建

* vue2创建项目我使用的是`vue create <project name>`

1. 我们使用自定义选项
![](http://ldmblog.ifoodin.com/20230717154242.png)

2. 选择我们需要的配置
![](http://ldmblog.ifoodin.com/20230717154326.png)

3. 路由模式不需要history，eslint选择默认
![](http://ldmblog.ifoodin.com/20230717154410.png)

4. 可以看到，`vue create`的**自定义**方式创建项目，默认使用`yarn`
![](http://ldmblog.ifoodin.com/20230717154630.png)

5. 一进入项目，我们可以看到已经为我们初始化了`git(master)`
![](http://ldmblog.ifoodin.com/20230717155438.png)

---

### Vue3项目创建

* vue3也可以用上面的创建方式，这里介绍另一种，这种是基于`create-vue`的，和上面的`vue create`不一样，这是用`vite`创建的项目

1. 创建命令为:
```bash
npm init vue@latest
```

2. 点击回车后，选择我们的配置
![](http://ldmblog.ifoodin.com/20230717160252.png)

*可以发现，用vite非常的快啊（*

3. 进入到项目中，我们可以发现并没有为我们初始化git，需要我们在终端中手动配置
```bash
$ git init 
$ git add .
$ git commit -m "init"
```

4. 然后我们需要开启`@`符对`src`的提示联想，只需在**项目根目录**下新建`jsconfig.json`文件，然后加上如下代码
```js
{
  "compilerOptions": {
    "baseUrl": "./",
    "paths": {
      "@/*": [
        "src/*"
      ]
    }
  }
}
```

5. 注意，这样做只是开启了联想，并**没有在底层进行路径转换**，不过在根目录下的`vite.config.js`已经给我们配好了
![](http://ldmblog.ifoodin.com/20230717161016.png)