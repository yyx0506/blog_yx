---
title: "ELK介绍"
date: 2022-04-28
categories:
- tranquilpeak
- features
tags:
- ELK
# keywords:
# - Elasticsearch
# - ELK
thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->

## ELK 的简单介绍

上一篇文章我们介绍了怎么安装es，下面我们来讲解elk


#### 1.1 elk 协议栈介绍及体系架构

![](/img/elkinfos1.png)

ELK 其实并不是一款软件，而是一整套解决方案，是三个软件产品的首字母缩写，Elasticsearch（ES），Logstash和 Kibana。这三款软件都是开源软件，通常是配合使用，而且又先后归于 Elastic.co 公司名下，故被简称为 ELK 协议栈。



Elasticsearch 是一个实时的分布式搜索和分析引擎，它可以用于全文搜索，结构化搜索以及分析。它是一个建立在全文搜索引擎 Apache Lucene 基础上的搜索引擎，使用 Java 语言编写。

主要特点

​       1 ：实时分析

​       2  ：分布式实时文件存储，并将每一个字段都编入索引

​       3  ：文档导向，所有的对象全部是文档

​       4  ：高可用性，易扩展，支持集群（Cluster）、分片和复制（Shards 和 Replicas）。见图 2 和图 3

​       5  ：接口友好，支持 JSON



Logstash 是一个具有实时渠道能力的数据收集引擎。使用 JRuby 语言编写。其作者是世界著名的运维工程师乔丹西塞 (JordanSissel)。

主要特点

​      1  :  几乎可以访问任何数据

​      2  :  可以和多种外部应用结合

​      3  ：支持弹性扩展

​      4   :  它由三个主要部分组成

​      5   ：Shipper－发送日志数据

​      6   :  Broker－收集数据，缺省内置 Redis

​      7   ：Indexer－数据写入

Kibana 是一款基于 Apache 开源协议，使用 JavaScript 语言编写，为 Elasticsearch 提供分析和可视化的 Web 平

台。它可以在 Elasticsearch 的索引中查找，交互数据，并生成各种维度的表图。

#### 1.2 elk整体架构


![](/img/elkinfos2.png)


#### 1.3 部分参考文档

ELK官网：https://www.elastic.co/

ELK官网文档：https://www.elastic.co/guide/index.html

ELK中文手册：https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html

ELK中文社区：https://elasticsearch.cn/

ELK API :https://www.elastic.co/guide/en/elasticsearch/client/java-api/current/travelansport-client.html