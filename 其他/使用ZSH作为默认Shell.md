# 使用ZSH作为默认Shell

> 因为需要`ssh`远程连接云服务器，而远端主机的`shell`没有高亮，实在太难受了，这篇文档就讲一下如何配置`zsh`作为默认`shell`，我这里就用我自己的`git bash`来演示，在`linux`主机上大同小异



### 一、将Git Bash整合进终端

我个人不喜欢用`xshell`来进行`ssh`连接，因为`xshell`实在太丑了（）

所以我会把`git`整合进`windows terminal`中，这样就好看多了，而且对于文本的操作也更友好

现在在安装`git`时，默认就有这个选项，勾选上即可，以前我还要手动配置（）

如果已经安装了`git`，那么可以参照这篇博客：[在windows Terminal中添加git_add git to terminal-CSDN博客](https://blog.csdn.net/michaelxuzhi___/article/details/106559172)





### 二、下载zsh

然后我们在这个网站：[Package: zsh - MSYS2 Packages](https://packages.msys2.org/package/zsh?repo=msys&variant=x86_64)，下载`zsh-5.9-2-x86_64.pkg.tar.zst`的软件压缩包

然后解压到你安装`git`的根目录下，有的文件会有冲突，我们直接选择覆盖原来的文件即可

![](http://ldmblog.ifoodin.com/20231004220459.png)



然后我们在终端输入`zsh`，出现以下内容说明安装完成

![](http://ldmblog.ifoodin.com/20231004174007.png)





### 三、安装oh-my-zsh

上面的输出就是让你开始配置`zsh`，但是我们自己纯手动配置太累了

所以我们下载`oh-my-zsh`，通过这个软件去配置，会轻松很多

我们输入如下命令进行安装

注意，安装的命令有可能会变动的，请以官网的为准：[Oh My Zsh - a delightful & open source framework for Zsh](https://ohmyz.sh/#install)

```sh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

然后我们等待，看到下面的输出就说明安装成功了，是不是一看就很漂亮（）

![](http://ldmblog.ifoodin.com/20231004174858.png)





### 四、安装oh-my-zsh插件

使用`oh-my-zsh`的好处，就是可以轻松配置插件，而且这些插件都非常好用



##### zsh-autosuggestions插件

这是命令自动补全插件，配置了这个插件后，`zsh`会自动记忆你输过的命令，给予补全提示

输入如下命令安装，安装到本地的`~/.oh-my-zsh/custom/plugins` 目录

```sh
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

然后在`~/.zshrc`中添加这个插件

```sh
# 编辑.zshrc
vim ~/.zshrc
```

在`plugins`中插入这个选项

![](http://ldmblog.ifoodin.com/20231004222636.png)



##### zsh-syntax-highlighting插件

这是一个语法校验和高亮插件，若指令不合法，则指令显示为红色，若指令合法就会显示为绿色

输入如下命令安装，同样是安装到本地的`~/.oh-my-zsh/custom/plugins` 目录

```sh
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting 
```

然后也是像上面那样配置`plugin`选项





### 五、配置oh-my-zsh主题

`oh-my-zsh`的可玩性很高，其中之一就是它的终端主题，非常多

具体的可以参考`oh-my-zsh`的主题官方文档：[Themes · ohmyzsh/ohmyzsh Wiki (github.com)](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes)

我用的主题不是官方自带的，叫做`Powerlevel10k`

使用后的效果大概是这样子的：

![](http://ldmblog.ifoodin.com/20231004215238.png)

输入如下命令安装该主题

```sh
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

然后在`~/.zshrc`文件中指定`ZSH_THEME`一项，然后退出编辑后按照指示来配置主题即可

![](http://ldmblog.ifoodin.com/20231004224013.png)





### 六、配置git默认终端为zsh

使用`vim`在`~/.bashrc`中加入如下内容即可

```sh
if [ -t 1 ]; then
  exec zsh
fi
```

![](http://ldmblog.ifoodin.com/20231004224258.png)

然后每次打开`git`就会以`zsh`作为默认`shell`了