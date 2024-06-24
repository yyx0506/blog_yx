---
title: "clickhouse 	简单介绍和搭建"
date: 2023-01-03
categories:
- tranquilpeak
- features
tags:
- clickhouse
# keywords:
# - clickhouse


thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->


### clickhouse 	简单介绍和搭建

#### 1 clickhouse 和 doris

```
Apache Doris是由百度贡献的开源MPP分析型数据库产品，亚秒级查询响应时间，支持实时数据分析；分布式架构简洁，易于运维，可以支持10PB以上的超大数据集；可以满足多种数据分析需求，例如固定历史报表，实时数据分析，交互式数据分析和探索式数据分析等。

ClickHouse是俄罗斯的搜索公司Yandex开源的MPP架构的分析引擎，号称比事务数据库快100-1000倍，团队有计算机体系结构的大牛，最大的特色是高性能的向量化执行引擎，而且功能丰富、可靠性高。

doris 的优点：使用起来相对简单一些 连接方式与mysql 类似 建表更简单，sql的标准支持更好，join的性能更好
clickhouse: 性能更佳，导入性能和单表查询性能更好，压缩数据更好，更省内存。具体要看场景使用不同的工具。

```

#### 2 clickhouse 和elk

```
目前大部分公司已经在将日志系统从logstash 转移到 clickhouse 上了，因为其及其优秀的压缩性能上，在大批量数据上，其性能十分的优异
ES 一定是最深入人心的日志系统了，它可以满足大数据量、全文检索的需求，但在成本与查询效率上很难平衡。ES 对成本过于敏感，配置低了查询速度会下降得非常厉害，保障查询速度又会大幅提高成本。
Grafana 推出的日志系统。理念上比较符合日志系统的需求，但现在还只是个玩具而已。
目前国内已知的是 uber 将日志全部换成了 clickhouse 当然他不只是作用于日志系统。

下面回顾一下基础概念：
OLTP：是传统的关系型数据库，主要操作增删改查，强调事务一致性，比如银行系统、电商系统。
OLAP：是仓库型数据库，主要是读取数据，做复杂数据分析，侧重技术决策支持，提供直观简单的结果。

ClickHouse做为列式数据库，列式数据库更适合OLAP场景，OLAP场景的关键特征：

而且相比于 doris 可以使用mysql 的链接工具 如 navicate 等连接 clickhouse 目前来说是使用dbeaver进行客户端v连接

```

 #### 3 安装clickhouse 

```
clickhouse 官方直接提供了一个线上的环境进行测试，我们直接使用docker-compose 来部署

首先创建一个clickhouse 的目录 
mkdir clockhouse 
然后进入到 clickhouse 后 
cd clickhouse
创建文件存储地址
mkdir db 


version: "3.6"

services:

  clickhouse-test:
    container_name: clickhouse-test
    image: yandex/clickhouse-server:latest
    restart: always
    ports:
      - "8123:8123"
      - "9000:9000"
      - "9009:9009"
    networks:
      - traefik
networks:
  traefik:
    external: true
```

直接启动使用 然后使用dbeaver 连接即可 

可以执行第一个命令  **show** **databases**