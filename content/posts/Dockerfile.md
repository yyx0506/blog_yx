---
title: "Dockerfile 以及 数据容器卷"
date: 2021-04-02
categories:
- tranquilpeak
- features
tags:
- docker
- dockerfile
# keywords:
# - docker
# - dockerfile

thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->

当我们需要去构建一个镜像 就需要使用到Dockerfile

基础知识

1:每个保留关键字（指令）都必须大写

2:执行从上到下顺序执行

3: # 表示注释

4:每个指令都会创建提交一个新的镜像层，并提交

### Dockerfile 例子

```
FROM  基础镜像，一切从这里构建  例如
# FROM python:3.8.5-slim FROM node:alpine
LABEL email="yyxyanyuxiang@163.com"
RUN 执行的命令 例如 
# RUN sed -i "s@http://deb.debian.org@http://mirrors.aliyun.com@g" /etc/apt/sources.list
# RUN yarn install
WORKDIR  镜像工作的地方 例如
# WORKDIR /opt/snapshot
VOLUME 设置卷挂载主机的目录
EXPOSE 设置暴露的端口 类似于docker run -p的动能
COPY 将文件拷贝到镜像中
ENV 设置环境变量
```

所有的镜像创建后要发布到dockerhub 或者其他docker仓库时需要 登陆

然后 使用docker build 命令 -t 意味着打标签 -f 用来指定dockerfile 的位置 指定镜像构建过程中的上下文环境的目录

dockerhub 是专门放docker 镜像的地址 可以自己申请一个账号，如果是公司层面的话 也可以去搭建一个harbor

```
docker build -t  xxxx/xxxx:xxx:0.0.1 . 创建镜像
docker push xxxx/xxxx:xxx:0.0.1   推送镜像到远程仓库
docker pull xxxx/xxxx:xxx:0.0.1   拉取镜像
对应服务器登陆docker 后直接拉取镜像即可
```

### 在Docker 中还有一个重要的东西叫做容器数据卷

将应用和环境打包成一个镜像！

如果数据都在容器中，那么如果容器删除，数据就会丢失！ 需求：数据可以持久化

mysql, 容器删了， 需求 mysql 数据可以存储在本地

容器之间可以有一个数据共享的技术！docker 容器中产生的数据同步到本地

这就是卷技术，是一种同步机制

为了容器的持久化和同步操作！容器间也是可以数据共享

主要是docker run -v 其中分为匿名挂载具名挂载和指定路径挂载

```
# 匿名挂载
docker run -d -p --name nginx01 -v /etc/nginx nginx

# 查看所有的卷的情况
docker volume ls
docker volume inspect 卷id

docker run -v 容器内路径 #匿名挂载
docker run -v 卷名：容器内路径 # 具名挂载
docker run -v 容器外路径：容器内路径 # 指定路径挂载 
可以在 容器内路径后加上 ：rw ro 等 设置容器权限
```

### 于此对应的还有一个容器数据卷

```
如果需要在容器之间共享一些数据，最简单的方法就是使用数据卷容器。数据卷容器是一个普通的容器，专门提供数据卷给其他容器挂载使用。
# 1 先创建一个数据卷容器
docker run -it -d --name centos01 -v /data1 -v /data2 centos bash
# 2 在创建一个数据卷容器
docker run -it -d --name centos02 --volumes-from v1 centos bash

```

#### 本篇内容到此结束