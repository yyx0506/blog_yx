---
title: "初识docker --一个神奇的工具"
date: 2022-03-27
categories:
- tranquilpeak
- features
tags:
- docker

# keywords:
# - docker

thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->


##Docker概述

Docker 是一个开源的应用容器引擎，基于 Go 语言 并遵从 Apache2.0 协议开源。

Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。

容器是完全使用沙箱机制，相互之间不会有任何接口（类似 iPhone 的 app）,更重要的是容器性能开销极低。

Docker 从 17.03 版本之后分为 CE（Community Edition: 社区版） 和 EE（Enterprise Edition: 企业版），我们用社区版就可以了。官网：https://docs.docker.com/

Docker 主要有三个部分组成 ： 镜像 容器 仓库

镜像可以又我们自己去创建也可以由他人创建后，我们自己拉取，容器是由镜像运行起来的，一个镜像可以运行多个容器，仓库就是存储镜像的地方。

## 第一步 : 在服务器上卸载docker 防止环境干扰

```
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine


# 如果还是卸载不掉的话 直接全删 yum -y remove docker*
```

## 第二步 : 安装需要的安装包（环境依赖）

```
yum install -y yum-utils
```

## 第三步 : 设置镜像的仓库 国外的源会很慢

```
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
# 可以替换为国内的源 例如清华或者阿里云（二选一即可）
sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

sudo yum-config-manager \
    --add-repo \
    https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo


```

## 第四步安装 : docker ce 是社区 ee是企业

```
yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

## 第五步 在 /etc/docker 下创建一个daemon.json文件 填入新的源

```
{

    "registry-mirrors":["https://almtd3fa.mirror.aliyuncs.com"]      

}
```

最后只需要service docker restart  就可以 通过docker --version (不建议通过 yum 直接安装docker 版本过低)



### 然后下一步就是如何使用docker 

#### 1:docker 帮助命令

```
docker version 显示docker版本信息
docker info 显示docker 的系统信息 包括 镜像和容器的数量
docker 命令 --help  帮助命令
```

#### 2:docker 镜像命令

```
docker searcg nginx filter=starts=6000  搜索nginx的镜像 并过滤超过6000点赞的的镜像版本
docker pull nginx 默认下载最新的版本
docker images 查看本地镜像
docker image prune -a -f  删除悬空的镜像
docker images -aq  获取所有镜像的id
docker rmi -f  $(docker images -aq) 删除所有镜像
docker rmi -f image_id 镜像id 删除指定的镜像
```

#### 3:docker容器命令 

```
docker run [可选参数] image 
# 参数说明
--name=="Name"  容器名字 tomcat1,tomcat2
-d 后台方式运行
-it 使用交互方式运行
-p 指定端口 8080:8080 主机端口映射到容器端口
-P 随机端口
-v /etc/localtime:/etc/localtime 时间选择
-v ./nginx.conf:/etc/nginx/nginx.conf 数据挂载
-e 配置环境 
docker ps 查看本地容器
docker ps -aq 获取本地容器的id
docker rm 容器id 删除指定容器
docker rm -f $(docker ps -aq) 删除全部容器
docker start 容器id 启动
docker stop 容器id 停止
docker restart  容器id 重启 
docker kill 容器id 杀掉容器
docker logs -f -t --tail 10 容器id 查看日志
docker top 容器id  docker 查看 容器内的进程
docker inspect 容器id/镜像id 显示容器/镜像详细信息
docker exec -it 0c5ea7f8954d /bin/bash
docker attach 0c5ea7f8954d 进入容器正在运行的终端
docker cp 容器id:容器内路径 容器外路径
```

### 以上就是docker 的介绍和简单运用，实际生产还要因不同场景而定