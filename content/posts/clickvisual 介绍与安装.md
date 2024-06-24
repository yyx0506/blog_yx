---
title: "clickvisual 介绍与安装"
date: 2023-02-01
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


### clickvisual 介绍与安装

#### 1什么是clickvisual

```
clickvisual 是一个轻量级的开源日志查询、分析、报警的可视化平台，致力于提供一站式应用可靠性的可视化的解决方案。既可以独立部署使用，也可作为插件集成到第三方系统。目前是市面上唯一一款支持 ClickHouse 的类 Kibana 的业务日志查询平台。

官方链接 https://clickvisual.gocn.vip/zh/

基本介绍都在上面 就不一一说明了

```

#### 2 什么场景可以去使用它

```
他是一个轻量级的开源日志 ，那么它对于日志的需求就相对简单 相比于主流的日志系统 其更加方便搭建
优点就是可以在前端查询 clickhouse 日志 可以配置一些简单的过滤 
缺点是 前端查询过滤功能比较弱，UI 也很简单 而且无法自定义日志查询命令
无法提供聚合函数，也无法通过聚合配置相关数据统计面板 前端无法提供数据导出功能

所以如果你的日志系统相对耦合度低 只是几个单一的日志表的话是可以使用此方案的

```



#### 3 搭建方式

```
我们使用docker-compose 来搭建系统

version: "3"
services:
  clickvisual:
    image: clickvisual/clickvisual:master
    container_name: clickvisual
    environment:
      EGO_CONFIG_PATH: /clickvisual/config/docker.toml
      EGO_LOG_WRITER: stderr
    ports:
      - "19001:19001"
    restart: always
    volumes:
      - ./config:/clickvisual/config
    command: [ '/bin/sh', '-c', './bin/clickvisual' ]
    
 直接启动即可
    docker-compose -f  clickvisual_compose.yml up -d 
    
    
实际使用 参考官方 文档即可 https://clickvisual.gocn.vip/clickvisual/01quickstart/experience-clickvisual-with-docker-compose.html#_2-%E6%93%8D%E4%BD%9C%E6%B5%81%E7%A8%8B
```

