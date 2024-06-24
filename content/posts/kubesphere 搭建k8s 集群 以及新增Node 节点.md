---
title: "kubesphere 搭建k8s 集群 以及新增Node 节点"
date: 2022-11-09
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

### kubesphere 搭建k8s 集群 以及新增Node 节点

#### 1: kubelet、kubeadm、kubectl 三者和k8s 的关系是什么

```
很多安装k8s 的教学都说的是直接 yum install 这三个 然后各种下一步操作就算安装好了k8s 
那么 这三个和k8s 的关系是什么

大家都知道k8s 是谷歌开源的一套容器化集群管理系统，是一个开源的容器编排引擎

kubeadm 是官方推出的一个用于快速部署 k8s 集群的工具（当然也有别的工具）

kubectl 是k8s 的集群的命令行工具 通过kubectl 能够对集群进行管理 ，并能够在集群上进行容器化应用的安装和部署 目前的指令大多是这个

kubelet 是master 派到node节点代表 管理本机容器，一个集群中每个节点上运行的代理，它保证了容器都运行在pod中


```

#### 2 :kubesphere 是什么

```
官方描述：KubeSphere 是在 Kubernetes 之上构建的面向云原生应用的分布式操作系统，完全开源，支持多云与多集群管理，提供全栈的 IT 自动化运维能力，简化企业的 DevOps 工作流。它的架构可以非常方便地使第三方应用与云原生生态组件进行即插即用 (plug-and-play) 的集成。

作为全栈的多租户容器平台，KubeSphere 提供了运维友好的向导式操作界面，帮助企业快速构建一个强大和功能丰富的容器云平台。KubeSphere 为用户提供构建企业级 Kubernetes 环境所需的多项功能，例如多云与多集群管理、Kubernetes 资源管理、DevOps、应用生命周期管理、微服务治理（服务网格）、日志查询与收集、服务与网络、多租户管理、监控告警、事件与审计查询、存储管理、访问权限控制、GPU 支持、网络策略、镜像仓库管理以及安全管理等。

KubeSphere 还开源了 KubeKey 帮助企业一键在公有云或数据中心快速搭建 Kubernetes 集群，提供单节点、多节点、集群插件安装，以及集群升级与运维。

```

#### 3 kubesphere 增加节点

```
目标路由 位于。ip:30880/clusters/default/nodes 下 可以查看节点信息
上一篇博客 我们使用了 一台服务器来部署 k8s 现在我们新增其他的服务器节点 

随意找一个 目录 执行一下 命令（个人是直接放在了root目录下）检索集群信息
./kk create config --from-cluster
然后会创建一个 sample.yaml 的配置文件
spec:
  hosts:
   - {name: master1, address: 00.168.1.56, internalAddress: 00.168.1.56}
   - {name: node1, address: 00.168.1.56, internalAddress: 00.168.1.56}  ## 新增节点 可自定义多个 
  roleGroups:
  roleGroups:
    etcd:
    - master1
    control-plane:
    - master1
    worker:
    - node1
    - node2


然后我们修改此处 添加新节点时，请勿修改现有节点的主机名。 用自己的主机名替换示例中的主机名。
有一个前提 服务器间可以互相通信 就需要将各自的公钥 加入到 knowhosts里  这个步骤就不细说了

然后执行一下命令 由于要下载所以会耗费一些时间 耐心等待 
./kk add nodes -f sample.yaml

如果出现某些 错误 比如超时  建议 yum -y update 更新一下相关库

删除节点 
./kk delete node <nodeName> -f config-sample.yaml


```



#### tips 下篇博客讲述 如何将服务部署在k8s上



