---
title: "Docker 可视化管理工具Portainer"
date: 2022-11-26
categories:
- tranquilpeak
- featureshu
tags:
- docker
- 博客
# keywords:
# - docker
# - Portainer

thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->


### Docker 可视化管理工具Portainer

**[官方链接](https://install.portainer.io/)**

#### 1: portioner 简介

```
Portainer是一个可视化的容器镜像的图形管理工具，利用Portainer可以轻松构建，管理和维护Docker环境。 而且完全免费，基于容器化的安装方式，方便高效部署。
官方安装说明：https://www.portainer.io/installation/
```

#### 2: 安装并使用

```
# 拉取镜像
docker pull portainer/portainer

# 查看镜像
docker images

# 启动容器
docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer

```

为什么需要映射两个端口呢？

- 8000端口是用来映射docker引擎的
- 9000端口是用来映射portainer web界面

为什么需要挂在两个文件

 ** 注意：此映射是必须要写的*

- -v /var/run/docker.sock:/var/run/docker.sock 为了保证portainer可视化页面和docker同步，所以我们需要把docker引擎和portainer可视化页面进行映射

 ***建议**

- -v /Users/lql/Desktop/docker_data/portainer_data:/data portainer/portainer

  portainer自身容器的一些数据挂在到宿主机。

最后登陆 到http://127.0.0.1:9000/ 查看信息 如下图所示

![](/img/portainer1.png)

设置个密码 nana123456789

#### 3 连接方式

```
可选择的模式有很多 比如 local 等
```

我们可以查看 images 的状态等  如下图所示

![](/img/portainer2.png)

这个工具相对来说比较简单 下去看看就ok了



