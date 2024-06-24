---
title: "Docker-compose 命名规范以及搭建redis"
date: 2022-10-11
categories:
- tranquilpeak
- featureshu
tags:
- docker
- redis
# keywords:
# - docker
# - docker-compose

thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->

### Docker 日常使用

### 1 一些命名规范

```
规范 一直是 程序猿的必备要求。好的规范 能够提高代码的可读性 以及后续维护性
docker 也不例外 

镜像的 名称 以及格式通常为 
DOCKER_REGISTRY/repo/name:tag，各个字段具体含义如下：

-----------------------基础镜像-----------------------
DOCKER_REGISTRY：企业统一的Docker Registry地址；
repo：镜像仓库，用来管理某一类镜像；
name：某个镜像的具体名称，一般的命名规则为：系统名称+系统版本+服务名+服务版本（如果公司约定了主要使用的系统名称和版本，则可以省略系统名称+系统版本部分，直接使用服务名作为镜像的名称）。例如：centos7.6-nginx-1.47。
tag：某个镜像具体的标签。例如：2.0。
需要注意的是：镜像的名称需要限制为[a-z0-9],其中可以出现的符号为[-._]，不能出现中文以及中文符号，包括镜像名称中的: 也必须是英文的冒号，不然创建容器的时候会失败。

然后这个冒号 在 镜像名称中只能出现一次（ 这是 一个大坑 ）

上文我们说到，我们采用一个独立的仓库来管理企业的基础镜像，例如使用public仓库来管理所有的基础镜像，下面我们来约定基础镜像的命名规则：DOCKER_REGISTRY/repo/name:tag；

repo：统一用public仓库来进行管理；

name：描述该image中所提供的软件，各软件间通过“-”连接；

tag：依次顺序描述该image中所提供的软件的版本，各版本间通过“-”连接。

-----------------------业务镜像-----------------------

业务镜像的命名规则以项目为仓库进行隔离，例如在一个支付项目中，我们使用payment镜像仓库，另一个风控项目使用risk-control镜像仓库，下面是我们约定的镜像命名规范：

repo：用项目名作为仓库，来管理该项目下的所有镜像。

name：描述该image中所包含的业务。

tag：commit id（前7位）和timestamp（12位，yymmddHHMMSS）组合成唯一标识，中间通过“-”连接。

当然了这个主要是 大开发团队中 才会出现的  


例如 yyx123456/nana:app_infos-1.0.2  第一个代表用户名 第二个代表仓库名 后面则是 业务名称和版本号

```

### 2 docker-compose 搭建 redis

```
先创建目录和配置文件
cd /usr/local
mkdir redis
cd redis/
touch docker-compose.yaml
touch redis.conf

然后编译 docker-compose.yaml 文件

version: "3.3"
services:
    redis:
        image: "redis:latest"
        container_name: redis-single
        restart: always
        ports:
          - "6379:6379"
        volumes:
          - ./redis.conf:/etc/redis/redis.conf:rw
          - ./data:/data:rw
        command:
          # 执行的命令
          redis-server /etc/redis/redis.conf --appendonly yes


再然后编译 redis.conf 文件
requirepass 123456
appendonly yes
#cluster-enable yes


然后测试连接可行   后续我们只需要 将 redis.conf 文件编写成自己可用的即可 比如密码的设置 以及 缓存的选择
一般来说 主要用到的 也就是下面的一些内容了。


     bind 0.0.0.0
     protected-mode yes
     port 6379
     tcp-backlog 511
     timeout 0
     tcp-keepalive 300
     daemonize  no
     supervised no
     pidfile /var/run/redis/redis.pid
     loglevel notice
     logfile /data/redis/redis.log
     databases 16
     always-show-logo yes
     save 900 1
     save 300 10
     save 60 10000
     stop-writes-on-bgsave-error yes
     rdbcompression yes
     rdbchecksum yes
     dbfilename dump.rdb
     dir /data/redis/
     slave-serve-stale-data yes
     slave-read-only yes
     repl-diskless-sync no
     repl-diskless-sync-delay 5
     repl-disable-tcp-nodelay no
     slave-priority 100
     maxmemory 495000000
     lazyfree-lazy-eviction no
     lazyfree-lazy-expire no
     lazyfree-lazy-server-del no
     slave-lazy-flush no
     appendonly no
     appendfilename "appendonly.aof"
     appendfsync everysec
     no-appendfsync-on-rewrite no
     auto-aof-rewrite-percentage 100
     auto-aof-rewrite-min-size 64mb
     aof-load-truncated yes
     aof-use-rdb-preamble no
     lua-time-limit 5000
     slowlog-log-slower-than 10000
     slowlog-max-len 128
     latency-monitor-threshold 0
     notify-keyspace-events ""
     hash-max-ziplist-entries 512
     hash-max-ziplist-value 64
     list-max-ziplist-size -2
     list-compress-depth 0
     set-max-intset-entries 512
     zset-max-ziplist-entries 128
     zset-max-ziplist-value 64
     hll-sparse-max-bytes 3000
     activerehashing yes
     client-output-buffer-limit normal 0 0 0
     client-output-buffer-limit slave 256mb 64mb 60
     client-output-buffer-limit pubsub 32mb 8mb 60
     hz 10
     aof-rewrite-incremental-fsync yes
```



