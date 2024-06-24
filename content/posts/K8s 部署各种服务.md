
---
title: "K8s 部署各种服务"
date: 2022-11-10
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

### K8s 部署各种服务 

#### 1 : kubesphere 添加应用商店

```
使用 admin 用户登录控制台，点击左上角的平台管理，选择集群管理。
点击定制资源定义，在搜索栏中输入 clusterconfiguration，点击结果查看其详细页面。
在自定义资源中，点击 ks-installer 右侧的 ，选择编辑 YAML。
在该 YAML 文件中，搜索 openpitrix，将 enabled 的 false 改为 true。完成后，点击右下角的确定，保存配置。
在 kubectl 中执行以下命令检查安装过程：
kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l 'app in (ks-install, ks-installer)' -o jsonpath='{.items[0].metadata.name}') -f
您可以通过点击控制台右下角的锤子按钮 找到 kubectl 工具。
安装需要一定的时间 
```

#### 2 kubesphere 部署应用

```
应用部署需要关注的信息【应用部署三要素】
1、应用的部署方式
2、应用的数据挂载（数据，配置文件）
3、应用的可访问性

部署有状态（）的服务 会稍显麻烦一点 

无状态服务
无状态服务不会在本地存储持久化数据.多个服务实例对于同一个用户请求的响应结果是完全一致的.这种多服务实例之间是没有依赖关系,比如web应用,在k8s控制器 中动态启停无状态服务的pod并不会对其它的pod产生影响.

有状态服务
有状态服务需要在本地存储持久化数据,典型的是分布式数据库的应用,分布式节点实例之间有依赖的拓扑关系.比如,主从关系. 如果K8S停止分布式集群中任 一实例pod,就可能会导致数据丢失或者集群的crash.比如数据库 一般来说很多时候是不建议 K8S  部署有状态服务的
```

下面使用 kubesphere 部署一个 mysql 

```
部署的前提是需要创建一个企业空间、一个项目和一个用户帐户 (project-regular)。该用户必须是已邀请至项目的平台普通用户，并且在项目中的角色为 operator。在本教程中，您需要以 project-regular 用户登录，并在 demo-workspace 企业空间的 demo-project 项目中进行操作。有关更多信息 请查看 https://kubesphere.io/zh/docs/v3.3/quick-start/create-workspace-and-project/

官方文档链接 ：https://kubesphere.io/zh/docs/v3.3/application-store/built-in-apps/mysql-app/ 

在 demo-project 的概览页面，点击左上角的应用商店。

找到 MySQL，在应用信息页面点击安装。

设置应用名称和版本，确保 MySQL 部署在 demo-project 项目中，然后点击下一步。

在应用设置页面，取消对 mysqlRootPassword 字段的注释并设置密码，然后点击安装。

等待 MySQL 创建完成并开始运行。

如何访问mysql 呢 

打开工作负载页面并点击 MySQL 的工作负载名称。

在容器组区域，展开容器详情，点击终端图标。

在终端窗口中，执行 mysql -uroot -ptesting 命令以 root 用户登录 MySQL。

从集群外访问 MySQL 数据库

要从集群外访问 MySQL，您需要先用 NodePort 暴露该应用。

打开服务页面并点击 MySQL 的服务名称。

点击更多操作，在下拉菜单中选择编辑外部访问。

将访问模式设置为 NodePort 并点击确定。有关更多信息，请参见项目网关。

您可以在端口区域查看暴露的端口。该端口号和公网 IP 地址将在下一步用于访问 MySQL 数据库。

您需要使用 MySQL Client 或第三方应用（例如 SQLPro Studio）才能访问 MySQL 数据库。以下演示如何使用 SQLPro Studio 访问 MySQL 数据库。

```



#### 3: k8s 的常用命令

```
查看节点状态：kubectl get nodes 
查看pods状态：kubectl get pods --all-namespaces
查看K8S集群状态: kubectl get cs
查看pods详情: kubectl get  pods名称
删除pods：kubectl delete pods名称
```



