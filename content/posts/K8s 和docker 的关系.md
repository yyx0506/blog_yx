
---
title: "K8s 和docker 的关系"
date: 2022-11-13
categories:
- tranquilpeak
- features
tags:
- k8s
# keywords:
# - k8s

thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->

### K8s 和docker 的关系

#### 1 : docker 出现与成长

```
大家都知道的是 docker 是由2010 年 一个叫 dotCloud的公司 开发的容器项目 后面因为docker 的发展也 改名为了docker公司

在互联网 迅速发展的那几年 大家所有的项目都是部署在自己公司的物理服务器上 ，这就导致了 当大体量服务器逐渐增多，包括对硬件场地维护成本的要求变得越来越高，如果没有备用服务器时很容易造成服务崩溃。
所以云服务器就应运而生，2006年亚马逊成立了aws 云存储业务开始滋生。
到2009年阿里云成立至今 已经改变大部分公司 的运行策略 开始使用云服务器进行 部署项目。

最开始 在paas 时代 vmware创立的Cloud Foundry是 当中最有名的，只不过后来在2021年已经死了。

那么容器是什么？ 容器是用来解决多个应用资源冲突与隔离型问题的技术。Linux上的namespace机制和cgroups命令都能用做资源隔离和限制，这些都是容器技术。容器技术并不是Docker创建的，在Docker兴起之前，就已经被其他公司商用了，但是为什么现在一谈起容器，所有人第一时间想到的就是Docker呢？这就要提到Cloud Foundry的死亡。

Cloud Foundry最核心的组件就是应用的打包和分发机制，也是开发者打交道最多的功能。Cloud Foundry为每一种主流的语言都定义了一套打包的方式，这些方式之间毫无章法。这就意味着 每一种语言 都不一样 ，这就导致了很容易出现问题，毕竟要适配每一种语言实在是太过于困难。
dokcer 怎么去解决 容器问题的呢 ，这就是大名鼎鼎的docker 镜像。没错 ，镜像run起来才是容器。
总之，Docker 镜像完美解决了两个问题：
1.本地环境和服务器环境的差异
2.同一份镜像可以让所有的机器进行复用

大家都知道镜像 可以在不同的服务器上进行 部署 它解决了一致性的问题 ，我们还需要面对复用性的问题 ，因此，Docker镜像使用了另一个技术：UnionFS以及一个全新的概念：层（layer），来优化每一个镜像的磁盘空间占用，提升镜像的复用性。这就是联合文件系统。
```

#### 2 docker发展与k8s 的诞生契机

```
了Docker项目利用自己创新的Docker Image瞬间爆红，那么其他互联网公司也想要去创建自己的镜像项目，CoreOS推出了Rocket（rkt）容器，Google也开源了自己的容器项目lmctfy，但是都没能竞争过docker ，随后docker公司为了自己的发展 推出了docker三件套
Docker Compose、Docker Swarm以及Docker Machine  前两者可能大部分人都听说过 特别是docker-compose

随着docker 公司的商业化，它严重侵害了曾经的合作公司RedHat的切身利益；再加之，Docker在2015年间的高速迭代表中现出了各种不稳定的breaking change开始让社区叫苦不迭。最终docker 公司迫于压力 和一些业内巨头联合成立了一个中立的基金会 runc
共同制定了一套容器和镜像的标准和规范 也就是 OCI ,OCI的成立，意味着容器运行时和镜像的实现与Docker项目完全剥离，让其他玩家不依赖Docker实现自己的Docker运行时成为可能。

由于docker公司对推进基金会的发展并不十分伤心 ，导致了oci基金会发展的十分的缓慢 ，因此 google redhat 等基础设施领域又共同牵头成立了CNCF 这也促进了k8s的诞生 与发展 。
```

#### 3 Kubernetes的出现与发展

```
k8s 是 google公司2014年发布开元的一个容器基础设施编排框架 k8s 与docker 的竞争也就拉开了帷幕 因为k8s开源开放的政策导致了 docker不得不承认失败 从2017年开始，Docker将Docker项目的容器运行时部分Containerd捐赠给了CNCF社区，并且在当年10月宣布将在自己的Docker企业版中内置Kubernetes项目，这也标志着持续了近两年的容器编排之战落下帷幕。

Docker构建的是以Docker容器为最核心的PaaS生态，包括以Docker Compose为主的简单容器关系编排，以及以Docker Swarm为主的线上运维平台。用户可以通过Docker Compose处理自己集群中容器之间的关系，并且通过Docker Swarm管理运维自己的集群，可以看到这一切其实就是当初Cloud Foundry的PaaS功能，所主打的就是和Docker容器的无缝集成。

所以就显得 docker 高度集中 而且他只能处理 容器之间的关系

而k8s 则不然 pod 的出现 让k8s 在容器编排时显得游刃有余 。 pod 是k8s 的运行应用的最小执行单位，跟docker最大的区别就是 
比如当你使用 grafana 和 Prometheus时 docker 是两个容器之间进行通信 那么我们需要处理两个容器间的通信关系，而pod则不需要，
pod 可以在一个容器内共享两个服务 他们可以使用相同的namespace 和cgroups 尽可能的减少资源调度

所以现阶段 在业务不太复杂的情况下都是直接使用 Docker。尽管 k8s 有很多好处，但是众所周知它非常复杂，业务比较简单可以放弃使用 k8s。
k8s 经常与 Docker 进行搭配使用，但是也可以使用其他容器，如RunC、Containerted（特别是2022年k8s 直接绕过了docker 放弃了docker 的代码）
```

#### 4 K8s 的容器编排

```
Kubernetes所做的容器编排核心内容其实是Pod编排，如何让这些Pod配合起来协同工作，则是编排的核心。在上一节中我们一起了解了kubernetes所做的是将各种关系进行了抽象，这些关系本质其实是Pod之间的关系。kubernetes将Pod的关系抽象成了以下几种，并且为这些关系定义了相对的控制器便于进行编排管理：

l  无状态Pod副本之间的协同关系——Deployment

l  有状态Pod副本之间的拓扑关系——StatefulSet

l  容器化守护进程——DaemonSet

l  离线业务——Job和CronJob

这些概念看起来可能让你有些不知所云，其实这些内容只是不同上述的控制器对Pod的不同的管理方式而已。

都知道 k8s 的部署文件是 以yaml 为主 具体 k8s 的文件编排 怎么去写 我们下期再讲
```

