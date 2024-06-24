---
title: "redis 键值设计"
date: 2022-06-06
categories:
- tranquilpeak
- features
tags:
- redis
# keywords:
# - redis

thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->


### redis 介绍4



redis 的底层结构  

```
当redis 服务器初始化时，会预先分配 16 个数据库
所有数据库保存到结构 redisServer 的一个成员 redisServer.db 数组中
redisClient中存在一个名叫db的指针指向当前使用的数据库
typedef struct redisDb { 
    int id; //id是数据库序号，为0-15（默认Redis有16个数据库） 
    long avg_ttl; //存储的数据库对象的平均ttl（time to live），用于统计 
    dict *dict; //存储数据库所有的key-value 
    dict *expires; //存储key的过期时间 
    dict *blocking_keys;//blpop 存储阻塞key和客户端对象 
    dict *ready_keys;//阻塞后push 响应阻塞客户端 存储阻塞后push的key和客户端对象 
    dict *watched_keys;//存储watch监控的的key和客户端对象 
} redisDb;

id               :    id是数据库序号，为0-15（默认Redis有16个数据库）
dict             :    存储数据库所有的key-value
expires          :   存储key的过期时间
RedisObject结构   :   Value是一个对象  包含字符串对象，列表对象，哈希对象，集合对象和有序集合对象

```

 

```
通信协议
Redis是单进程单线程的  应用系统和Redis通过Redis协议（RESP）进行交互
Redis协议位于TCP层之上，即客户端和Redis实例保持双工的连接
```

#### 使用建议

##### 1: 键值设计

```
key名称的设计 可读性和可管理性 以业务名(或数据库名)为前缀(防止key冲突)，用冒号分隔，比如业务名:表名:id
比如 string:user:id   可以简写为 s:u:id 
list:app:user 可以简写为 l:a:u  set:app:user h:key:app  等
```

##### 2:命令使用

```
例如hgetall、lrange、smembers、zrange、sinter等并非不能使用，但是需要明确N的值。有遍历的
需求可以使用hscan、sscan、zscan scan 代替。  (一般来说50w一下的数据可以用)
使用批量操作提高效率
1.原生命令：例如mget、mset。
2.非原生命令：可以使用pipeline提高效率。
两者不同：
1.原生是原子操作，pipeline是非原子操作。
2.pipeline可以打包不同的命令，原生做不到
3.pipeline需要客户端和服务端同时支持。
删除 key
1、Hash删除: hscan + hdel
List删除: ltrim Set删除: sscan + srem SortedSet删除: zscan + zrem
```









