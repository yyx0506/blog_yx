---
title: "Screen -- 跑一次行脚本的好工具"
date: 2022-05-02
categories:
- tranquilpeak
- features
tags:
- screen
- linux
# keywords:
# - screen
# - linux

thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->


### Screen -- 跑一次行脚本的好工具

#### 为什么要用screen ？

当我们有一个程序只需要跑一遍，然而在本地跑又太慢，部署到服务器的话使用后台挂载又太麻烦，能不能把他放到一个黑窗口里自己跑，然后我可以随时切换出去，不耽误我的服务器做其他事？远程服务器的时候，断网或者手误关掉了远程终端，会导致会话中断程序终止。而Screen连接的终端，会话独立运行，程序会一直进行。而且会话可以恢复，还可以自行删除。所以有了这个小工具

#### 常用命令

```
yum install screen             # 安装screen 
screen -S yourname             # 新建一个叫yourname 的session
screen -ls                     # 列出当前所有的session
screen -r yourname             # 回到yourname这个session
screen -d yourname						 # 远程detach某个session  快捷键是 ctrl a+d
screen -d -r yourname 				 # 结束当前session并回到yourname这个session
screen -S yourname -X quit     # 删除这个叫yourname 的session
```





