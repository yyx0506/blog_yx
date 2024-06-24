---
title: "Docker 总结"
date: 2023-01-01
categories:
- tranquilpeak
- featureshu
tags:
- docker
# keywords:
# - docker
thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->

### Docker 总结

#### 1: 什么是docker

```
Docker 是一个开源的应用容器引擎，基于go 语言开发并遵循了apache2.0 协议开源
Docker 是在Linux 容器里运行应用的开源工具，是一种轻量级的“虚拟机”
Docker 的容器技术可以在一台主机上轻松为任何应用创建一个轻量级的，可移植的，自给自足的容器

```

#### 2:Docker的应用场景有哪些？

```
Web 应用的自动化打包和发布。
自动化测试和持续集成、发布。
在服务型环境中部署和调整数据库或其他的后台应用。
从头编译或者扩展现有的 OpenShift 或 Cloud Foundry 平台来搭建自己的 PaaS 环境。
在这里我重点介绍下Docker作为内部开发环境的场景
在容器技术出现之前，公司往往是通过为每个开发人员提供一台或者多台虚拟机来充当开发测试环境。开发测试环境一般负载较低，大量的系统资源都被浪费在虚拟机本身的进程上了。
Docker容器没有任何CPU和内存上的额外开销，很适合用来提供公司内部的开发测试环境。而且由于docker镜像可以很方便的在公司内部分享，这对开发环境的规范性也有极大的帮助。
如果要把容器作为开发机使用，需要解决的是远程登录容器和容器内进程管理问题。虽然docker的初衷是为“微服务”架构设计的，但根据我们的实际使用经验，在docker内运行多个程序，甚至sshd或者upstart也是可行的。
```

#### 3: Docker的优点有哪些

```
容器化越来越受欢迎，Docker的容器有点总结如下：
灵活：即使是最复杂的应用也可以集装箱化。
轻量级：容器利用并共享主机内核。
可互换：可以即时部署更新和升级。
便携式：可以在本地构建，部署到云，并在任何地方运行。
可扩展：可以增加并白动分发容器副本。
可堆叠：可以垂直和即时堆叠服务。
Docker 是一个用于开发，交付和运行应用程序的开放平台。Docker 使您能够将应用程序与基础架构分开，从而可以快速交付软件。借助 Docker，您可以与管理应用程序相同的方式来管理基础架构。通过利用 Docker 的方法来快速交付，测试和部署代码，您可以大大减少编写代码和在生产环境中运行代码之间的延迟。
```

#### 4:Docker与虚拟机的区别是什么？

```
虚拟机通过添加Hypervisor层（虚拟化中间层），虚拟出网卡、内存、CPU等虚拟硬件，再在其上建立虚拟机，每个虚拟机都有自己的系统内核。而Docker容器则是通过隔离（namesapce）的方式，将文件系统、进程、设备、网络等资源进行隔离，再对权限、CPU资源等进行控制（cgroup），最终让容器之间互不影响，容器无法影响宿主机。
与虚拟机相比，容器资源损耗要少。同样的宿主机下，能够建立容器的数量要比虚拟机多
但是，虚拟机的安全性要比容器稍好，而docker容器与宿主机共享内核、文件系统等资源，更有可能对其他容器、宿主机产生影响。
```

#### 5:Docker的三大核心是什么？

```
--------------------镜像--------------------
Docker的镜像是创建容器的基础，类似虚拟机的快照，可以理解为一个面向Docker容器引擎的只读模板。
通过镜像启动一个容器，一个镜像是一个可执行的包，其中包括运行应用程序所需要的所有内容包含代码，运行时间，库、环境变量、和配置文件。
Docker镜像也是一个压缩包，只是这个压缩包不只是可执行文件，环境部署脚本，它还包含了完整的操作系统。因为大部分的镜像都是基于某个操作系统来构建，所以很轻松的就可以构建本地和远端一样的环境，这也是Docker镜像的精髓。
--------------------容器--------------------
Docker的容器是从镜像创建的运行实例，它可以被启动、停止和删除。所创建的每一个容器都是相互隔离、互不可见，以保证平台的安全性。可以把容器看做是一个简易版的linux环境（包括root用户权限、镜像空间、用户空间和网络空间等）和运行在其中的应用程序。
--------------------仓库--------------------
仓库注册服务器上往往存放着多个仓库，每个仓库中包含了多个镜像，每个镜像有不同标签（tag）。
仓库分为公开仓库（Public）和私有仓库（Private）两种形式。
最大的公开仓库是 Docker Hub:https://hub.docker.com，存放了数量庞大的镜像供用户下载。
国内的公开仓库包括阿里云 、网易云等。
```

#### 6:如何快速安装Docker？

```
执行以下安装命令去安装依赖包
第一步卸载旧的docker 
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine


# 如果还是卸载不掉的话 直接全删 yum -y remove docker*
第二步需要的安装包
yum install -y yum-utils
第三步 设置镜像的仓库 国外的源会很慢
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
可以改为国内的源 (阿里云)
sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
第四步安装docker ce 是社区 ee是企业
yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
在 /etc/docker 下创建一个daemon.json文件 填入新的缘
{ "registry-mirrors":["https://almtd3fa.mirror.aliyuncs.com"]      }
然后service docker restart 重启docker服务
```

#### 7:如何修改Docker的存储位置？

```
默认情况下 Docker的存放位置为：/var/lib/docker
可以通过命令查看具体位置：docker info | grep “Docker Root Dir”
修改到其它目录
首先停掉 Docker 服务：
systemctl stop docker
然后移动整个/var/lib/docker 目录到目的路径
mkdir -p /root/data/docker
mv /var/lib/docker /root/data/docker
ln -s /root/data/docker /var/lib/docker --快捷方式
```

#### 8:Docker 镜像常用管理有哪些？

```
快速检索镜像
格式：docker search 关键字
获取镜像
格式：docker   pull   仓库名称[:标签] 如果下载镜像时不指定标签，则默认会下载仓库中最新版本的镜像，即选择标签为 latest 标签
查看镜像信息

镜像下载后默认存放在 /var/lib/docker
REPOSITORY: 镜像所属仓库  TAG: 镜像的标签信息，标记同一个仓库中的不同镜像 IMAGE ID ：镜像的唯一ID号，唯一标识一个镜像
CREATED: 镜像创建时间  SIZE: 镜像大小
docker images 查看本地镜像
为本地镜像添加新的标签
格式：docker   tag  名称:[ 标签]
删除镜像  格式1：docker   rmi   仓库名称:标签 当一个镜像有多个标签时，只是删除其中指定的标签格式2: docker   rmi  镜像ID  [-f]  如果该镜像已经被容器使用，正确的做法是先删除依赖该镜像的所有容器，再去删除镜像

格式2: docker   rmi  镜像ID  [-f]  如果该镜像已经被容器使用，正确的做法是先删除依赖该镜像的所有容器，再去删除镜像
将镜像保存为本地文件  格式：docker   save   -o  存储文件名   存储的镜像
```

#### 9:Docker网络模式有哪些？

```
host模式  host 模式 ：使用 --net=host 指定
相当于VMware 中的桥接模式，与宿主机在同一个网络中，但是没有独立IP地址
Docker 使用了Linux 的Namespace 技术来进行资源隔离，如PID Namespace隔离进程，Mount Namespace隔离文件系统，Network Namespace 隔离网络等。
一个Network Namespace 提供了一份独立的网络环境，包括网卡，路由，iptable 规则等都与其他Network Namespace 隔离。
一个Docker 容器一般会分配一个独立的Network Namespace
但是如果启动容器的时候使用host 模式，那么这个容器将不会获得一个独立的Network Namespace ，而是和宿主机共用一个Network Namespace 。容器将不会虚拟出自己的网卡，配置自己的IP等，而是使用宿主机的IP和端口.此时容器不再拥有隔离的、独立的网络栈。不拥有所有端口资源

container模式
container模式：使用–net=contatiner:NAME_or_ID 指定
这个模式指定新创建的容器和已经存在的一个容器共享一个Network Namespace，而不是和宿主机共享。新创建的容器不会创建自己的网卡，配置自己的IP，而是和一个指定的容器共享IP，端口范围等。 可以在一定程度上节省网络资源，容器内部依然不会拥有所有端口。
同样，两个容器除了网络方面，其他的如文件系统，进程列表等还是隔离的。
两个容器的进程可以通过lo网卡设备通信

none 模式
none模式:使用 --net=none指定
使用none 模式，docker 容器有自己的network Namespace ，但是并不为Docker 容器进行任何网络配置。也就是说，这个Docker 容器没有网卡，ip， 路由等信息。
这种网络模式下，容器只有lo 回环网络，没有其他网卡。
这种类型没有办法联网，但是封闭的网络能很好的保证容器的安全性
该容器将完全独立于网络，用户可以根据需要为容器添加网卡。此模式拥有所有端口。（none网络模式配置网络）特殊情况下才会用到，一般不用

bridge 模式
相当于Vmware中的 nat 模式，容器使用独立network Namespace，并连接到docker0虚拟网卡。通过docker0网桥以及iptables nat表配置与宿主机通信，此模式会为每一个容器分配Network Namespace、设置IP等，并将一个主机上的 Docker 容器连接到一个虚拟网桥上。
当Docker进程启动时，会在主机上创建一个名为docker0的虚拟网桥，此主机上启动的Docker容器会连接到这个虚拟网桥上。虚拟网桥的工作方式和物理交换机类似，这样主机上的所有容器就通过交换机连在了一个二层网络中。
从docker0子网中分配一个IP给容器使用，并设置docker0的IP地址为容器的默认网关。在主机上创建一对虚拟网卡veth pair设备。veth设备总是成对出现的，它们组成了一个数据的通道，数据从一个设备进入，就会从另一个设备出来。因此，veth设备常用来连接两个网络设备。
Docker将veth pair 设备的一端放在新创建的容器中，并命名为eth0（容器的网卡），另一端放在主机中， 以veth*这样类似的名字命名，并将这个网络设备加入到docker0网桥中。可以通过 brctl show 命令查看。
容器之间通过veth pair进行访问
使用 docker run -p 时，docker实际是在iptables做了DNAT规则，实现端口转发功能。
可以使用iptables -t nat -vnL 查看。
```

#### 10:什么是Docker的数据卷

```
数据卷是一个供容器使用的特殊目录，位于容器中。可将宿主机的目录挂载到数据卷上，对数据卷的修改操作立刻可见，并且更新数据不会影响镜像，从而实现数据在宿主机与容器之间的迁移。数据卷的使用类似于Linux下对目录进行的mount操作。
如果需要在容器之间共享一些数据，最简单的方法就是使用数据卷容器。数据卷容器是一个普通的容器，专门提供数据卷给其他容器挂载使用。
容器互联是通过容器的名称在容器间建立一条专门的网络通信隧道。简单点说，就是会在源容器和接收容器之间建立一条隧道，接收容器可以看到源容器指定的信息
```

#### 11:如何搭建Docker私有仓库

```
直接选择harbor
```



