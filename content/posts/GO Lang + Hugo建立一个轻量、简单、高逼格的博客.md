---
title: "GO Lang + Hugo建立一个简单的博客并使用gitpages进行部署"
date: 2018-04-27
categories:
- tranquilpeak
- features
tags:
- Hugo
- 博客
# keywords:
# - disqus
# - google
# - gravatar
thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->

## GOLang+Hugo建立一个简单的博客并使用gitpages进行部署

Go lang是谷歌2009年发布的开源编程语言，截止目前go的release版本已经到了1.10。go语言的开发人员都是计算机界大神一般的存在，go语言目前可以达到c/c++80%的性能，远快于c/c++的编译速度，目前很火的开源软件docker、kubernetes、lxd等软件都是使用go语言编写的，而且2016年Go语言被评为年度编程语言，可见go的应用场景非同一般。。

先看看市面上大部分 个人博客站点的选择是什么：

 WordPress：大而繁，高度定制，插件巨多。

Typecho：小而巧，该有的都有，与WordPress相比较，相当于恐龙与麻雀的区别。

Gridea：简单粗暴，专注写作。

Hexo：配置麻烦。

Halo：基于java，吃内存，docker部署最方便。

Hugo：基于go，渲染快，官网号称最快。

为什么选择hugo ,配置简单（作者很懒）支持markdown 可以静态部署，方便简洁。

#### 首先，可以去go的官网网站下载安装包 https://golang.org/dl/ 然后直接双击安装即可，不需要配置环境变量，因为安装过程自动配置，安装完毕后，打开命令行，输入

```python
go version
```

#### 显示主版本号即表示安装成功

#### 方法2 使用 brew下载 go（mac电脑工作者的福音）

~~~
brew install go
~~~

#### 然后再下载 hugo

~~~pytohn
brew install hugo
~~~

#### 完以后，在命令行内输入

~~~
hugo version
~~~

#### 打印出版本号即表示hugo安装成功



## 创建博客

#### 在命令行输入

~~~
hugo new site hugo_blog
~~~

#### 就生成了一个名字为hugo_blog的新站点 

#### 如下图

![](/img/bolg.png)

#### 然后进入thenmes目录，下载自己喜欢的主题

#### 更多免费开源的主题可以在这里下载 https://themes.gohugo.io/

### 如下图是我在thenmes目录下pull下来的主题

![](/img/bolg_3.png)

#### 我们需要把主题exampleSite目录下面的 static、config.toml、content 复制到 hugo_blog目录下进行替换



#### 然后再命令行中输入

~~~
hugo server
~~~

#### 如下图 可以看到已经在1313端口起了一个hugo服务

![](/img/bolg_4.png)



#### 访问一下

![](/img/bolg_5.png)



#### 至此，非常快速而简单博客已经做好了，那么如何部署到线上呢？非常简单输入命令进行打包操作

~~~ 
hugo --baseUrl="/"
~~~

#### hugo就会把你的站点生成纯静态页面，然后打包到public文件夹

然后下一步最重要的是更改目录下面的config.toml文件，主要是这个baseurl的值需要更改

首先第一步就是去个人的码云仓库新建一个仓库，然后选择公开仓库，最后在服务中选择gitpages服务，将照片信息上传等待，成功以后gitee会给到我们一个地址，将这个地址写入到config.toml文件的baseurl中即可，然后将代码上传选择gitpages服务更新即可。