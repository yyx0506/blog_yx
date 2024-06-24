---
title: "关于iview"
date: 2023-02-28
categories:
- tranquilpeak
- features
tags:
- ivew
- vue


# keywords:
# - 队列


thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->



### 关于iview

#### 1: 什么是iview

```
官方 https://iview.github.io/
iView 是一套基于 Vue.js 的开源 UI 组件库，主要服务于 PC 界面的中后台产品。
特性 
      高质量、功能丰富
      友好的 API ，自由灵活地使用空间
      细致、漂亮的 UI
      事无巨细的文档
      可自定义主题

安装 :使用 npm
npm install iview --save
```

#### 2: 开始搭建后台

```
第一步 idea puglins 安装vue.js

然后创建一个空项目。拉取下来后 直接 npm install 生成一个node 项目

首先没有安装node 
下载安装包（选择macOS 安装包 (.pkg)：下载 | Node.js 中文网
直接点击下载包一直无脑点下一步安装
查看版本号，检测是否安装成功：npm -v
安装cnpm：
sudo npm install -g cnpm --registry=https://registry.npm.taobao.org

安装vue-cli 脚手架：
sudo npm install -g @vue/cli
然后 
vue -V 查看 vue版本 
创建一个新的文件夹

vue create test

创建项目时 选择预设 ，按上下键选择Manually select features 回车
选择项目设置， 按上下键切换 按空格键选择或者取消选择 ，选择Babel,Router,Vuex,CSS Pre-processors然后按回车键
选择Vue.js的版本 选者 3.x
选择history模式 如果是就输入Y
择css预处理器，选择less回车
选择设置存放，询问你是把设置另外放在一个文件夹里 还是放在package.json文件夹里，回车



然后在 main.js中 引入ivew
import iView from 'iview'
Vue.use(iView)

最后我们直接 npm run serve 启动项目即可


```

#### 3: 关于npm 和node

```
Node.js是JavaScript的一种运行环境
node.js是对Google V8引擎进行的封装。是一个服务器端的javascript的解释器
npm是node.js的包管理器 类似于 gomod 对于go 的作用
所以我们一般拉一个项目 第一件事就是npm install 就是下载缺少的依赖等
npm不需要单独的安装，在安装Node的时候会将npm一起安装
所以我们只需要安装一个 node即可
```

#### 4:开始搭建页面

```
所需要的组件直接查看ivew 官网即可 https://iview.github.io/
```









