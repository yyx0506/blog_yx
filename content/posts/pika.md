---
title: "pika介绍"
date: 2022-04-27
categories:
- tranquilpeak
- features
tags:
- pika
# keywords:
# - pika

thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->

### pika 

```
众所周知 redis 的性能很好 ，但是内存太贵 ，如果将redis 的模式移植到 硬盘上 是不是可以 牺牲一部分性能来降低费用呢
所以就出现了 pika 以及tendis 这次就先讲一下 pika
```

#### pika的搭建

```
老样子 我们选择 docker-compose 搭建
首先写一写配置文件 如下


version: '3'
services:
  pika_9001:
    container_name: pika_9001
    image: pikadb/pika:v3.3.5
    restart: always
    ports: 
      - "9001:9001"
    volumes:

      - "/etc/localtime:/etc/localtime:ro"
      - ./conf:/pika/conf
      - ./data:/pika/data
    logging:
      driver: "json-file"
      options:
        max-size: "200k"
        max-file: "10"

然后创建两个文件夹 data 以及 conf

其中data 是 数据所在地 conf 是 pika 的配置文件

```

#### conf 配置文件

```
# Pika port   Pika 端口
port : 9001  
# Thread Number   pika进程数量，不建议超过核心数量，pika是多线程的
thread-num : 1
# Thread Pool Size  sync 处理线程的任务队列大小，一般没有必要修改
thread-pool-size : 12
# Sync Thread Number Sync 线程数量
sync-thread-num : 6
# Pika log path  Pika日志目录
log-path : ./log/
# Pika db path  Pika数据目录
db-path : ./db/
# Pika write-buffer-size  Pika 底层引擎的write_buffer_size配置，大，会快，但越大刷盘越久，需要权衡，实际上在测试中发现再大意义也不大了
write-buffer-size : 268435456
# Pika timeout  Pika 的连接超时时间，就是连接sleep多久了就把它断开
timeout : 60
# Requirepass  密码管理员密码,默认为空
requirepass :
# Masterauth
masterauth :
# Userpass 用户密码,默认为空
userpass :
# User Blacklist
userblacklist :
# if this option is set to 'classic', that means pika support multiple DB, in
# this mode, option databases enable
# if this option is set to 'sharding', that means pika support multiple Table, you
# can specify slot num for each table, in this mode, option default-slot-num enable
# Pika instance mode [classic | sharding]
instance-mode : classic
# Set the number of databases. The default database is DB 0, you can select
# a different one on a per-connection basis using SELECT <dbid> where
# dbid is a number between 0 and 'databases' - 1, limited in [1, 8]
databases : 1
# default slot number each table in sharding mode
default-slot-num : 1024
# replication num defines how many followers in a single raft group, only [0, 1, 2, 3, 4] is valid
replication-num : 0
# consensus level defines how many confirms does leader get, before commit this log to client,
#                 only [0, ...replicaiton-num] is valid
consensus-level : 0
# Dump Prefix
dump-prefix :
# daemonize  [yes | no]
#daemonize : yes
# Dump Path
dump-path : ./dump/
# Expire-dump-days
dump-expire : 0
# pidfile Path
pidfile : ./pika.pid
# Max Connection
maxclients : 20000
# the per file size of sst to compact, default is 20M
target-file-size-base : 20971520
# Expire-logs-days
expire-logs-days : 7
# Expire-logs-nums
expire-logs-nums : 10
# Root-connection-num
root-connection-num : 2
# Slowlog-write-errorlog
slowlog-write-errorlog : no
# Slowlog-log-slower-than
slowlog-log-slower-than : 10000
# Slowlog-max-len
slowlog-max-len : 128
# Pika db sync path
db-sync-path : ./dbsync/
# db sync speed(MB) max is set to 1024MB, min is set to 0, and if below 0 or above 1024, the value will be adjust to 1024
db-sync-speed : -1
# The slave priority
slave-priority : 100
# network interface
#network-interface : eth1
# replication
#slaveof : master-ip:master-port

# CronTask, format 1: start-end/ratio, like 02-04/60, pika will check to schedule compaction between 2 to 4 o'clock everyday
#                   if the freesize/disksize > 60%.
#           format 2: week/start-end/ratio, like 3/02-04/60, pika will check to schedule compaction between 2 to 4 o'clock
#                   every wednesday, if the freesize/disksize > 60%.
#           NOTICE: if compact-interval is set, compact-cron will be mask and disable.
#
#compact-cron : 3/02-04/60

# Compact-interval, format: interval/ratio, like 6/60, pika will check to schedule compaction every 6 hours,
#                           if the freesize/disksize > 60%. NOTICE:compact-interval is prior than compact-cron;
#compact-interval :

# the size of flow control window while sync binlog between master and slave.Default is 9000 and the maximum is 90000.
sync-window-size : 9000
# max value of connection read buffer size: configurable value 67108864(64MB) or 268435456(256MB) or 536870912(512MB)
#                                           default value is 268435456(256MB)
#                                           NOTICE: master and slave should share exactly the same value
max-conn-rbuf-size : 268435456


###################
## Critical Settings
###################
# write_binlog  [yes | no]
write-binlog : yes
# binlog file size: default is 100M,  limited in [1K, 2G]
binlog-file-size : 104857600
# Automatically triggers a small compaction according statistics
# Use the cache to store up to 'max-cache-statistic-keys' keys
# if 'max-cache-statistic-keys' set to '0', that means turn off the statistics function
# it also doesn't automatically trigger a small compact feature
max-cache-statistic-keys : 0
# When 'delete' or 'overwrite' a specific multi-data structure key 'small-compaction-threshold' times,
# a small compact is triggered automatically, default is 5000, limited in [1, 100000]
small-compaction-threshold : 5000
# If the total size of all live memtables of all the DBs exceeds
# the limit, a flush will be triggered in the next DB to which the next write
# is issued.
max-write-buffer-size : 10737418240
# Limit some command response size, like Scan, Keys*
max-client-response-size : 1073741824
# Compression type supported [snappy, zlib, lz4, zstd]
compression : snappy
# max-background-flushes: default is 1, limited in [1, 4]
max-background-flushes : 1
# max-background-compactions: default is 2, limited in [1, 8]
max-background-compactions : 2
# maximum value of Rocksdb cached open file descriptors
max-cache-files : 5000
# max_bytes_for_level_multiplier: default is 10, you can change it to 5
max-bytes-for-level-multiplier : 10
# BlockBasedTable block_size, default 4k
# block-size: 4096
# block LRU cache, default 8M, 0 to disable
# block-cache: 8388608
# whether the block cache is shared among the RocksDB instances, default is per CF
# share-block-cache: no
# whether or not index and filter blocks is stored in block cache
# cache-index-and-filter-blocks: no
# when set to yes, bloomfilter of the last level will not be built
# optimize-filters-for-hits: no
# https://github.com/facebook/rocksdb/wiki/Leveled-Compaction#levels-target-size
# level-compaction-dynamic-level-bytes: no
```

#### 详细介绍

```
# Pika 端口
port : 9221

# pika进程数量，不建议超过核心数量，pika是多线程的
thread-num : 1

# Sync 线程数量
sync-thread-num : 6

# sync 处理线程的任务队列大小，一般没有必要修改
sync-buffer-size : 10

# Pika日志目录
log-path : ./log/

# Pika 的log级别，任何一个级别均记录慢日志
loglevel : info

# Pika数据目录
db-path : ./db/

# Pika 底层引擎的write_buffer_size配置，大，会快，但越大刷盘越久，需要权衡，实际上在测试中发现再大意义也不大了
write-buffer-size : 268435456

# Pika 的连接超时时间，就是连接sleep多久了就把它断开
timeout : 60

# 密码管理员密码,默认为空
requirepass : password

# Masterauth
masterauth :

# 用户密码,默认为空
userpass : userpass
 
# 指令黑名单,普通用户将不能使用黑名单中的指令。指令之间使用“,”隔开。默认为空
userblacklist : FLUSHALL,SHUTDOWN

# Pika的dump文件名称前缀
dump-prefix : pika9001-
 
# 守护进程模式  [yes | no]
daemonize : yes
 
# slotmigrate  [yes | no]
#slotmigrate : no

# Pika dump目录
dump-path : /data1/pika9001/dump/

# pidfile Path pid文件目录
pidfile : /data1/pika9001/pid/9001.pid
 
# Max Connection 
maxconnection : 20000
 
# rocks-db的sst文件体积，sst文件是层级的，文件越小，速度越快，合并代价越低，但文件数量就会超多，而文件越大，速度相对变慢，合并代价大，但文件数量会很少，默认是 20M
target-file-size-base : 20971520

# write2file文件保留时间，7天，最小为1，超过7天的文件会被自动清理
expire-logs-days : 7
 
# write2file文件最大数量，200个，最小为10，超过200个就开始自动清理，始终保留200个
expire-logs-nums : 200
 
# root用户连接保证数量：2个，即时Max Connection用完，该参数也能确保本地（127.0.0.1）有10个连接可以同来登陆pika
root-connection-num : 2
 
# 慢日志记录时间，单位为微秒
slowlog-log-slower-than : 10000

# slave是否是只读状态(yes/no, 1/0)
# slave-read-only : 0

# Pika db 同步路径
db-sync-path : ./dbsync/

# db sync speed(MB) max is set to 125MB, min is set to 0, and if below 0 or above 125, the value will be adjust to 125
db-sync-speed : -1

# 指定网卡
# network-interface : eth1
# replication
# slaveof : master-ip:master-port

###################
## Critical Settings
###################
# # write2file文件体积，默认为100MB，一旦启动不可修改,  limited in [1K, 2G]
binlog-file-size : 104857600

# 压缩方式[snappy | none]默认为snappy，一旦启动不可修改
compression : snappy

# 指定后台flush线程数量，默认为1，范围为[1, 4]
max-background-flushes : 1

# 指定后台压缩线程数量，默认为1，范围为[1, 4]
max-background-compactions : 1

# max-cache-files default is 5000
max-cache-files : 5000
```







