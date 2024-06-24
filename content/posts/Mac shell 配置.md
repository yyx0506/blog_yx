---
title: "Mac shell 配置"
date: 2023-04-11
categories:
- tranquilpeak
- features
tags:
- mac
# keywords:
# - linux
# - grep
# - crontab
# - netstat
# - df

thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->


### Mac shell 配置

### 1:myzsh

```
本文只以 MacOS 为例，而其他 OS 的安装方式，请仔细阅读以上工具的文档。

使用Linux的小伙伴都知道默认shell是bash 恰巧mac 是基于Unix内核的图形化操作系统
Linux 和 Unix 是不同的，但它们之间确实存在关系，因为 Linux 是从 Unix 派生的。
所以二者在命令上有很大的相似 甚至一般用户感知不出来具体的差别
zsh的作为一种shell，功能更强大，但是配置复杂。后来，有大佬创建了一个名为oh-my-zsh的开源项目，使用方便，功能强大。
下面就去oh-my-zsh的配置。
```

##### 1.1 安装 zsh

想要安装oh-my-zsh,首先得安装zsh，安装zsh输入以下命令

brew install zsh 

chsh -s /bin/zsh

执行以上命令安装 zsh，再重启终端即可发现 Shell 变成了 zsh。

zsh --version  可以查看版本情况

##### 1.2 安装ohmyzsh

方式一：使用curl

sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

方式二：使用wget

sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"

##### 1.3 安装 starship

brew install starship

执行以上命令安装 starship，然后把 `eval "$(starship init zsh)"` 写入到 `~/.zshrc` 文件末尾。 再执行 `$SHELL` 就可以看到 Shell 变漂亮了～

但没法像这个 demo 一样，因为有 [fish Shell](http://fishshell.com/) 的加持，有将历史命令以阴影的方式展现展现出来，命令自动补全等功能。 这就需要安装相应的插件，接下来需要安装的 antibody ，正是用来安装管理这些组件的。

##### 1.4 安装 antibody

brew install getantibody/tap/antibody

执行以上命令安装 antibody，然后创建 `~/.zsh_plugins.txt`。 它保存着需要安装的 zsh 插件，每行都是一个 Github Repos 名称。

```plain
zsh-users/zsh-completions
zsh-users/zsh-history-substring-search
zsh-users/zsh-syntax-highlighting
zsh-users/zsh-autosuggestions
```

再把以下内容写入到 `~/.zshrc` 文件末尾。

source <(antibody init) 

antibody bundle < ~/.zsh_plugins.txt

执行 `$SHELL` 或者重新打开终端即可生效。 更改 `~/.zsh_plugins.txt` 后，首次打开 Shell 会安装不存在的插件。稍微等待一下，即可享受这颜值与实力并存的生产力工具了。

具体如何使用 antibody 直接看其文档https://getantibody.github.io/

如何安装也可以查看 官方文档 https://github.com/ohmyzsh/ohmyzsh/wiki/Installing-ZSH

##### 1.5 主题设置

**查看oh-my-zsh自带的所有主题，输入如下命令**

 ls ~/.oh-my-zsh/themes

更改主题。

*vim* ~/.zshrc

可以看到 ZSH_THEME="robbyrussell"

我们可以使用上述的主题进行直接替换

**你还可以设置为random，表示每次打开编辑器随机选一种主题，当你想查看当前主题可以输入以下命令查看**

echo $ZSH_THEME

##### 1.6 相关的插件

```
ls ~/.oh-my-zsh/plugins
```

*vim* ~/.zshrc

定位到plugins位置， 可以查看哪些插件开启了

**推荐插件**

**alias**

这是自带插件，用来查看所有的git命令，直接用以下命令添加到plugins中可以开启插件

source ~/.oh-my-zsh/plugins/git/git.plugin.zsh

### **incr**

这个插件必不可少，用来代码自动补全，安装命令如下：

```
wget http://mimosa-pudica.net/src/incr-0.2.zsh
mkdir ~/.oh-my-zsh/plugins/incr
mv incr-0.2.zsh ~/.oh-my-zsh/plugins/incr
echo 'source ~/.oh-my-zsh/plugins/incr/incr*.zsh' >> ~/.zshrc
source ~/.zshrc
```

