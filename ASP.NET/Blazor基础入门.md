# Blazor基础入门

在学习之前我有些话想对自己说，`blazor`一个很冷门的框架，我之前在暑假的时候已经接触过

但是学习资料太少，微软的文档很多人觉得写的很好，但我觉得非常的晦涩、混乱

不知道是不是我水平太差，无法理解，所以我一直没太弄明白`blazor`这个框架

我自己也很少尝试纯看文档，不看视频，甚至连博客都没得看的学习方式

所以我在这里立下一个`flag`，我必把`blazor`和其他`.net`知识给啃下来，至少做到能够全栈开发一些小项目的水平





### 一、先决条件

> 哈哈哈，这个`先决条件`是不是很熟悉，这就是微软文档经常使用的话术，下面我们看看在使用`.net`之前需要做什么准备工作



##### 1. 安装.NET SDK

这里去`.net`官网安装即可，建议安装`.net 6.0`，因为是长期支持版

链接：https://dotnet.microsoft.com/zh-cn

安装完成后输入命令检查一下是否成功安装：

![](http://ldmblog.ifoodin.com/20230919164105.png)



##### 2. 安装IDE

我的老师（也可以说是朋友哈哈）推荐我用`Jet Brains`的`rider`，我觉得比`vs`好看

想用哪一个自己决定就好了





### 二、初识Blazor Server项目

> 首先我们先来用`dotnet cli`来创建一个`blazor`项目，然后解析一下项目的结构和一些核心语法



##### 1. 创建blazor server项目

输入以下命令：

* `-o`表示输出的目录名称

* `-f`表示指定的`.net`的版本

```sh
dotnet blazorserver -o blazor-server-demo -f net6.0
```



##### 2. 把项目跑起来

初次启动，可能会看见浏览器这个报错：

![](http://ldmblog.ifoodin.com/20230919163302.png)

这是因为我们还未在本地信任`SSL`证书，所以我们在终端输入如下命令：

```sh
dotnet dev-certs https --trust
```

然后会弹出如下窗口：

![](http://ldmblog.ifoodin.com/20230919165710.png)

我们点击“是”，然后重启`blazor`项目，可以看到初始化的项目：

![](http://ldmblog.ifoodin.com/image-20230919170131968.png)



##### 3. 项目结构分析

首先我们要弄清楚`blazor`里面三种最常规的文件：`.cshtml`、`.razor`、`.cs`

* `.razor`是组件，可以理解为`vue`中的`.vue`组件，可以在里面写`html`、`c#`、`css`

* `.cshtml`是页面文件，和`.razor`类似，一般我们写新建文件都是`.razor`，很少用`.cshtml`，这个一般是创建项目自带的
* `.cs`就是`c#`代码了，用于实现一些逻辑操作



使用`rider`打开项目后我们可以看到如下项目结构，我们根据官方文档来解读一下：

![](http://ldmblog.ifoodin.com/20230919164722.png)



`pages`文件夹是存放页面和组件的

从中我们不难看出`Index`、`FetchData`、`Couter`三个`razor`组件分别对应我们浏览器中看到的三个侧边栏选项

并且它们的文件中都有`@page`的指令，对应着不同的路径



那么包装这三个`razor`组件的组件，又是哪个呢？

没错，有个非常显眼的`razor`组件--`MainLayout.razor`，看名字也应该看出来了吧

`MainLayout.razor`就是应用整体包装组件--也就是**布局组件**

可以看到这个布局组件继承了`LayoutComponentBase`基类

并且在布局组件中，使用`@Body`指令可以渲染其他的`razor`组件

![](http://ldmblog.ifoodin.com/20230919191015.png)



讲完了布局组件，那么应用肯定还有一个**根组件**对吧

这个和`vue`如出一辙，名字就叫做`App.razor`

在`blazor`项目中，它的路由是和`vue`不一样的，`blazor`的路由**只在根组件中配置一次**，然后就可以解析整个应用的路由匹配了

然后分别用`<Found>`和`<NotFound>`标签来配置匹配到路由和没有匹配到路由时的返回结果

然后我们在根组件中可以指定布局组件，这里默认布局组件就是`MinLayout.razor`

![](http://ldmblog.ifoodin.com/20230919191543.png)



到了根组件，你以为就完了吗，这就跟`vue`不一样了，`blazor`中还有一个**根页面**`_Host.cshtml`

可以看到这里有一个`<component>`标签，这就是**根页面指定根组件**用的

在`blazor`中，所有的页面请求都会先导航到`_Host.cshtml`页面

然后如果有匹配到，就跳转对应的页面/组件，如果没有匹配到，就匹配`<NotFound>`标签中的内容

还有就是我试过了，`_Host`里面的`@page`的路径可以随便写，但最好还是不要改了吧，避免出错

![](http://ldmblog.ifoodin.com/20230919220621.png)



除了根页面，还有一个根页面的布局页面`_Layout.cshtml`

![](http://ldmblog.ifoodin.com/20230919223215.png)

我们在根页面中看到的这一段，就是在指定根页面的布局页面

这一段代码可以让`_Host.cshtml`继承`_Layout.cshtml`布局页面，然后其中的内容就可以通过`@RenderBody()`来渲染

```csharp
@ {
    Layout = "_Layout";
}
```



这大概就是`blazor`的文件结构分析，然后我根据自己的理解作了一张关系图，可以看一下：

![](http://ldmblog.ifoodin.com/_Host.cshtml.png)



### 三、开发Blazor应用的基础知识及概念

> 我在开篇说过，`blazor`的资料太少了，所以想要从头到尾纯看文档不太现实，我们不妨联系一下以前学过的东西，这里我就以`vue`还有`java`作为类比，来学习`blazor`的知识点



##### 1. 编写c#代码逻辑

这一点就和`vue`中的`script`一样，用于编写交互逻辑

我们只需要在`html`标签部分下使用`@code`指令包裹`c#`代码即可

```csharp
@code {
	// c#代码逻辑...
}
```



##### 2. 在html中嵌套c#代码

这个就跟`vue`的指令差不多，只不过我感觉这个更类似于`react`，而不是像`vue`那样将指令写在`html`标签的属性位置上

我们使用`@`符号，然后后面跟上`c#`代码即可，如果是代码块，还可以用大括号包裹

下面展示几种常用的写法：

更全的讲解请查看文档：[ASP.NET Core 的 Razor 语法参考 | Microsoft Learn](https://learn.microsoft.com/zh-cn/aspnet/core/mvc/views/razor?view=aspnetcore-7.0)

```csharp
// 插值语法
<div>number: @value<div/>

// 条件渲染
@if (num == 1) {
	<p>num等于1<p/>    
}else {
	<p>num不等于1<p/>   
}

// 列表渲染
@foreach (var item in array) {
    <div>@item<div/>
}

// 动态绑定类名
// 其他的条件属性同理
<div calss="@NavMenuCssClass"><div/>
@code {
    private bool collapseNavMenu = true;
    private string? NavMenuCssClass => collapseNavMenu ? "collapse" : null;
}
```



##### 3. 事件绑定

在这里只举几个简单的例子，详情请查看微软的文档：[ASP.NET Core Blazor 事件处理 | Microsoft Learn](https://learn.microsoft.com/zh-cn/aspnet/core/blazor/components/event-handling?view=aspnetcore-7.0)



`blazor`事件绑定和`vue`的比较类似，都是`on{EVENT}`的写法

```html
// 绑定点击事件
<button @onclick="IncrementCount">Click me</button>
```

此外，我们还可以传入事件参数

```csharp
<button @onclick="IncrementCount">Click me</button>

@code {
	public void IncrementCount(MouseEventArgs e) {
		Console.WriteLine(e);
	}
}
```

那如果我们想在传入事件参数的时候携带别的参数呢？

这个时候就要用到`lambda`表达式了

因为`blazor`事件绑定的**委托**函数只能传一个事件参数，所以我们在委托函数内指定我们的事件处理函数，再传参即可

```csharp
<button class="btn btn-primary" @onclick="@(e => IncrementCount(e, 3))">Click me</button>

@code {
    private void IncrementCount(MouseEventArgs e, Int32 num) {
        Console.WriteLine(e);
    }
}
```



##### 4. 获取组件实例

这个也是像`vue`那样，用`@ref`给组件打个标记，然后在`c#`代码中声明该实例的类型即可

```csharp
<input @ref="exampleInput" />

@code {
    // 声明元素实例
    private ElementReference exampleInput;
} 
```

然后我们就可以使用该元素的一些内置方法及属性，例如`input`框聚焦

```csharp
private async Task ChangeFocus() {
    await exampleInput.FocusAsync();
}
```



##### 5. 组件间通信

`blazor`组件间通信有多种方式，我们逐一讲解看一下



* 组件参数传递：

这个就是`vue`中的组件`props`，在`blazor`中我们用`[Parameter]`声明要接收的属性

```csharp
// 子组件
@code {
    [Parameter]
    public string? Title { get; set; }
}
```

父组件向子组件传递`Parameter`

要注意，传参的时候"="两边**不能有空格**

```html
// 父组件
<ChildCompnent Title="I'm your father">
```



* EventCallback

这就相当于`vue`中的`emit`，子组件出发父组件的事件并且传参给父组件

这里子组件定义的`EventCallback<T>`类型的泛型参数`T`需要和父组件的事件处理函数一致

```csharp
// 子组件
<button @onclick="@(e => OnClickCallback.InvokeAsync(8))">点我向父组件传递消息</button>

@code {
    [Parameter]
    public EventCallback<Int32> OnClickCallback { get; set; }
}



// 父组件
<ChildComponent OnClickCallback="@changeCnt"/>
    
@code {
    private Int32 cnt = 0;

    // 传给子组件的事件处理函数
    private void changeCnt(Int32 num)
    {
        this.cnt = num;
    }
}
```



* @attributes指令

这个我感觉我用到的地方应该不多，但官方文档又讲了，就简单过一下

`@attributes`就是向子组件传入自定义属性

```csharp
<div @attributes="AdditionalAttributes"/>

@code {
    [Parameter(CaptureUnmatchedValues = true)]
    public IDictionary<string, object>? AdditionalAttributes { get; set; }
}
```

可以看到传进来的属性是一个字典，如果不理解可以再看一个例子：

就是一个字典中有多个多个**键值对**，这些键值对就是属性和值

```csharp
<input @attributes="InputAttributes" />

@code {
    private Dictionary<string, object> InputAttributes { get; set; } = new() {
        { "maxlength", "10" },
        { "placeholder", "Input placeholder text" },
        { "required", "required" },
        { "size", "50" }
    };
}
```



* RenderFragment

这个其实就是`vue`中的插槽

我们只需要在`razor`的`[Parameter]`属性中指定属性类型为`RenderFragment`即可，然后子组件就可以使用该属性渲染`dom`了

```csharp
// 父组件
<ChildComponent>
    <p>传给子组件的内容<p/>
<ChildComponent>

// 子组件
<div>@FragmentContent<div/>
    
@code {
    [Parameter]
    private RenderFramgment FrangmentContent { get; set; }
}
```





##### 5. 数据绑定

这里只作简单介绍，详情请查看文档：[ASP.NET Core Blazor 数据绑定 | Microsoft Learn](https://learn.microsoft.com/zh-cn/aspnet/core/blazor/components/data-binding?view=aspnetcore-6.0)



对于文本框的输入，我们也可以像`vue`那样做数据绑定

```csharp
<input @bind="InputValue" />

@code {
    private string? InputValue { get; set; }
}
```

上面那种写法是默认在`blur`时会修改`InputValue`字段的值

如果我想要一边输入一边修改或者其他的修改方式呢？

我们可以再加上一个`@bind:event="on{event}"`的指令属性来指定

例如这样，我们就可以在`input`的时候更新`InputValue`的值了

```html
<input @bind="InputValue" @bind:event="oninput" />
```



##### 6. 路由与导航

这里仍旧是只讲最常用的部分，详情请查看官方文档：[ASP.NET Core Blazor 路由和导航 | Microsoft Learn](https://learn.microsoft.com/zh-cn/aspnet/core/blazor/fundamentals/routing?view=aspnetcore-7.0)



对于`blazor`中的路由，因为其概念与`vue`的路由不一样，我更愿意称其为**导航**，避免弄混

`blazor`应用的路由只配置在`App.razor`中，并且只配置一次，不是像`vue`那样的每个路由组件都要引入并配置好路径

然后只要在新建组件时使用`@page`指令来指定路由组件的路径即可

![](http://ldmblog.ifoodin.com/20230919191543.png)



* 路由参数

在`blazor`中，路由参数是由`[Parameter]`注解声明的，然后就可以在路由`url`中传入参数了

```csharp
// 设置路由参数占位符，并且参数是可选的
@page "/test/{param?}"

@code {
    // 使用注解声明为参数
    [Parameter]
    private string param { get; set; }
}
```



* 查询字符串

`blazor`的路由`url`中，也可以使用查询字符串来作为参数

我们需要将`[SupplyParameterFromQuery]`和`[Parameter]`注解作一个结合

通过`Name`属性，我们还可以**将查询参数名指定为我们自定义的属性名**

```csharp
[Parameter]
[SupplyParameterFromQuery(Name = "{查询参数名}")]
private string <自定义对应查询参数的属性名> { get; set; }
```



* 编程式导航

这个也和`vue`一样，但是我们需要先注入`NavigationManager`服务（注入服务会放在后面讲，这里只是简单提一下，不影响理解）

```csharp
// 注入服务
@inject NavigationManager Navigation

<button @onClick="handleNavigate">点我跳转<button/>

@code {
    public void handleNavigate() {
        Navigation.NavigateTo("path");
    }
}
```



* 导航拦截

在`blazor`导航里面貌似没有取消导航到某个路径的操作，我看了半天文档，也问了ai，就是没找到

所以就先简单写一下吧，参照官方文档即可

在`App.razor`根组件中编写如下代码

```csharp
<Router AppAssembly="@typeof(App).Assembly"
    	// 设置OnNavigateAsync委托
        OnNavigateAsync="@OnNavigateAsync">
    // 路由配置...
</Router>
    
@code {
    private async Task OnNavigateAsync(NavigationContext context)
    {
        // 对某个路径进行操作
        if (context.Path == "index")
        {
            Navigation.NavigateTo("");
        }
    }
}
```



##### 7. 组件指令

使用指令是`blazor`的一大特点，下面来看几个常用的

有些指令是`.cshtml`页面的，有些是`.razor`页面的，这里我们就只说`.razor`页面的哈

详情参照官方文档：[ASP.NET Core 的 Razor 语法参考 | Microsoft Learn](https://learn.microsoft.com/zh-cn/aspnet/core/mvc/views/razor?view=aspnetcore-6.0)



* `@attribute`指令：

这个指令一般用于`@attribute [Authorize]`，表示该组件必须要授权的用户才可以访问，关于用户授权会在后面说



* `@implements`指令：

用于实现某个接口，例如这里我们实现`IDisposable`接口

实现了这个接口后，`blazor`在组件销毁时就会自动调用`Dispose`方法，但是具体的逻辑需要我们实现

```csharp
@implements IDisposable
    
@code {
    public void Dispose() {
        // 释放资源
    }
}
```



##### 8. 页面布局

其实在`blazor`中，**布局**才是相当于`vue`中的路由

`vue`中的嵌套路由，在`blazor`中是用嵌套布局实现的



我们先参照一下`Mainlayout.razor`

在这里面，有一个`@inherits LayoutComponentBase`指令，这就表示该`razor`组件是作为布局组件

![](http://ldmblog.ifoodin.com/20230919191015.png)

在布局组件中，我们让然可以指定**该布局组件的布局组件**，这就是嵌套布局

比如说我有一个`Test.razor`，它是一个布局组件，但是它又指定了它的布局组件为`MainLayout.razor`

```csharp
// Test.razor
@inherits LayoutComponentBase
@layout Mainlayout
    
<div>@Body<div/>
```

在别的非布局组件中，就可以用`@layout`指令来指定它的布局组件是哪一个了



##### 9. 组件生命周期

这里也是讲用得多的`hook`，详情请参考官方文档：[ASP.NET Core Razor 组件生命周期 | Microsoft Learn](https://learn.microsoft.com/zh-cn/aspnet/core/blazor/components/lifecycle?view=aspnetcore-6.0#when-parameters-are-set-setparametersasync)



和`vue`一样，`blazor`中也有生命周期`hook`

在`blazor`中，如果要使用生命周期函数，在`.razor`组件中重写生命周期`hook`即可

参考微软官方文档的流程图：

![](http://ldmblog.ifoodin.com/lifecycle1.png)



* `Oninitialized{Async}`

这个相当于`vue`里面的`created`生命周期，就是组件初始化的时候会调用这个`hook`，此时组件的参数还未设置好

这个`hook`会在每次显示该组件--也就是路由到该组件的时候就会触发

```csharp
@code {
    protected override void OnInitialized()
    {
        Console.WriteLine("index组件初始化啦");
    }
}
```

这个生命周期有时会**触发两次**，微软的官方文档有如下说明：

![](http://ldmblog.ifoodin.com/20230921203332.png)

我测试过，如果该组件为加载时**最初呈现在浏览器页面中（也就是首页）**的话，那么就会触发两次

所以在最初渲染的页面中，尽量避免使用这个`hook`吧

所以微软也给我们设置了另一个生命周期，就是我们下面要讲的那个



* `OnAfterRender{Async}`

这个就相当于`vue`里面的`onMounted`了，此时所有的参数都已经加载好了

我们还可以利用这个声明周期函数自带的一个参数：`firstRender`

这个参数用于判断该组件是否是首次渲染，只有首次加载该组件时该值才会为`true`

```csharp
@code {
    protected override void OnAfterRender(bool firstRender)
    {
        if (firstRender)
        {
            Console.WriteLine("组件首次渲染");
        }
    }
}
```



