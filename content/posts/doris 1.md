
---
title: "doris 介绍"
date: 2022-08-02
categories:
- tranquilpeak
- features
tags:
- doris
- 博客
# keywords:
# - doris
# - fe
# - be
thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->
### doris 1

#### 1:doris 介绍

```
Apache Doris是一个现代化的MPP分析型数据库产品。
亚秒级响应时间即可获得查询结果
并且可以支持10PB以上的超大数据集
Doris 兼容 MySQL 的协议与语法，使用任何 MySQL 客户端和常用 BI 工具都可以对接Doris来进行数据分析
Doris 支持高效的明细数据分析，并且支持针对明细数据建立多重 Rollup。Rollup是 Doris 中⼀种利用预计算进行查询加速的技术，可以显著提升查询性能和查询并发。
```

#### 2:doris 特性

```
Doris的架构设计融合了传统MPP数据库，以及Hadoop生态分布式系统的设计思想，具有以下独特性：
1、现代化 MPP 架构 (Massively Parallel Processing  大规模并行处理) 
2、秒级查询返回延时 
3、支持标准SQL语言，兼容MySQL协议 
4、向量化执行器 
5、高效的聚合表技术 
6、新型预聚合技术Rollup 
7、高性能、高可用、高 
8、极简运维，弹性伸缩
```

#### 3:doris 架构

```
Doris 集群的正常运行完全不需要依赖任何其他系统。用户通过部署两类进程 FE(Frontend),
BE(Backend) 就能够搭建⼀个完整的 Doris 集群。Doris的极简架构降低了 Doris 的管理复杂度。管理
员只需要专注于 Doris 系统，无需学习和管理任何其他外部系统。
在 Doris 架构中，FE 主要负责集群元数据的管理；BE 主要负责集群数据的管理，以及执行所有的查询
计划。FE、BE 均可水平扩展。
⼀个典型的用户查询请求处理流程如下：
1、FE接受用户发送过来的查询请求； 
2、FE会把这个查询请求执行查询规划，生成可以执行的执行计划； 
3、FE把这个执行计划分发给需要执行的BE节点； 
4、BE分别将对应的执行计划处理完毕； 
5、BE将各个查询的结果汇总到执行计划指定的BE上； 
6、完成结果汇总的BE将结果返回给FE； 
7、FE把查询最后的结果发送给客户端。
Doris 采用分布式架构，FE，BE 节点都可以水平扩展。Doris 集群的规模可以从 1 个节点扩展到几百个
节点，支持的数据规模能够达到10PB 级别。Doris 支持快速的实时导入以及批量导入功能，单机的导
入吞吐性能能够达到 100MB/s。 Doris既能够支持大查询，也能够支持高并发查询模式。Doris 能够支
持亚秒级查询，并且查询 QPS 可达 10000 以上
```

#### 4:doris 的安装

```
https://doris.apache.org/zh-CN/docs/get-starting/ 官方文档地址
docker pull apache/doris:build-env-for-0.15.0
docker run -it apache/doris:build-env-for-0.15.0
建议以挂载本地 Doris 源码目录的方式运行镜像，这样编译的产出二进制文件会存储在宿主机中，不会因为镜像退出而消失。
同时，建议同时将镜像中 maven 的 .m2 目录挂载到宿主机目录，以防止每次启动镜像编译时，重复下载 maven 的依赖库。
```

