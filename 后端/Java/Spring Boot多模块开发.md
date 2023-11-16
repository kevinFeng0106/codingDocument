# Spring Boot多模块开发

在一般的开发中不太可能只是单一模块的项目了，而且我自己也没有搭建过多模块的`spring boot`项目

那么这篇文档就来总结一下如何搭建

注意：我使用的是`java8`+`spring boot2`的组合，如果使用`java17`，请用`spring boot3`，否则多模块项目中会出现一些问题



`spring boot`多模块项目搭建主要分以下几步：

* 搭建基本的模块结构
* 处理父模块与子模块之间的依赖关系
* 处理各子模块之间的依赖关系



### 一、搭建基本结构

我们先把基本的结构搭起来，再去处理各模块间的关系



##### 1. 创建一个spring boot项目（父模块）

这就不用多说了吧，设置好`GroupId`和`ArtifictId`，就跟单模块的时候一样

然后因为这是父模块，不需要在里面写代码，所以需要把`src`文件夹和其他一些没用的`maven`文件夹删除掉



##### 2、创建两个子模块

也是像创建单模块项目那样创建，但是`GroupId`和`spring boot`的版本需要和父模块保持一致，`ArtifictId`随便起就行

然后也是把没用的文件删掉，但是不要删除`src`文件夹，因为我们要在子模块中写业务代码



搭建完之后的基本结构如下图所示，一个父模块下包含了`module1`和`module2`两个子模块

![](http://ldmblog.ifoodin.com/20231108124909.png)





### 二、处理父子模块的依赖关系

父模块是用来统一管理子模块的依赖的，所以我们首先配置好它们之间的关系



##### 1. 配置父模块的pom.xml

在父模块的`pom.xml`添加如下配置

```xml
<!-- 将主包的打包格式设为pom包（默认为jar包） -->
<packaging>pom</packaging>

<!-- 设置它的子模块有哪些 -->
<modules>
    <module>module1</module>
    <module>module2</module>
</modules>
```



##### 2. 配置子模块的pom.xml

将子模块原本的`<parent>`标签中的指向`spring boot`的内容替换为指向当前项目的父模块的内容

```xml
<!--  设置父模块  -->
<parent>
    <groupId>com.lordmoon</groupId>
    <artifactId>spring-multi-mod</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <relativePath>../pom.xml</relativePath> // 引用父模块的pom.xml文件
</parent>
```

然后父子模块关系其实就配置好了，此时在父模块中添加的依赖，在子模块中就可以直接使用了，不用再在子模块的`pom.xml`引入





### 三、处理子模块间的依赖关系

实际开发时，某些子模块肯定要用到另一些子模块中的东西，所以我们还需要配置子模块间的依赖



##### 1. 删除非核心子模块的配置文件和启动类

因为我们不是微服务，所以只需要一个启动类，所以我们仅需要在一个核心子模块中保留启动类和`yml`配置文件

直接手动删除即可



##### 2. 在需要的子模块中配置dependency

我这里是在`module1`中依赖了`module2`，就直接像引入外部依赖那样配置即可

```xml
<dependencies>
    <!--  配置子模块间的依赖关系(依赖module2)  -->
    <dependency>
        <groupId>com.lordmoon</groupId>
        <artifactId>module2</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </dependency>
</dependencies>
```





### 四、多模块的Bean扫描和配置

因为不只有一个模块，而启动类和`yml`核心配置文件又只有一个，所以如何让核心模块扫描到别的模块的`bean`非常重要

当我们按照功能划分模块时，一般每个模块中都会包含各自的`controller`、`service`、`mapper`层

那么如何配置才能使得每个模块的各个层都正常工作呢？



##### 1. 在核心模块添加对其他模块的依赖

当你要在其他模块使用三层结构的时候，首先要在核心模块中添加对这些模块的依赖，否则就无法找到对应的包



##### 2. 在核心模块的启动类添加对应注解

因为启动类默认只能扫描本包下的`bean`，所以它默认是无法扫描其他模块的`bean`的

所以我们添加上`@ComponentScan`和`@MapperScan`的注解来扫描其他模块

![](http://ldmblog.ifoodin.com/20231117001330.png)

这样在别的模块中就可以正常使用三层结构了