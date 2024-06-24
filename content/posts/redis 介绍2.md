---
title: "redis 缓存 架构的设计思路  事务"
date: 2022-06-04
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

### redis 介绍2

#### 1:缓存的使用场景

```
DB缓存，减轻DB服务器压力:
一般情况下数据存在数据库中，应用程序直接操作数据库。
当访问量上万，数据库压力增大，可以采取的方案有：
读写分离，分库分表
当访问量达到10万、百万，需要引入缓存。
将已经访问过的内容或数据存储起来，当再次访问时先找缓存，缓存命中返回数据。
不命中再找数据库，并回填缓存
```

#### 2:缓存有三种读写模式

```
1:Cache Aside Pattern（常用）:
Cache Aside Pattern（旁路缓存），是最经典的缓存+数据库读写模式。
读的时候，先读缓存，缓存没有的话，就读数据库，然后取出数据后放入缓存，同时返回响应。
更新的时候，先更新数据库，然后再删除缓存
为什么是删除缓存，而不是更新缓存呢？
1、缓存的值是一个结构：hash、list，更新数据需要遍历
先遍历（耗时）后修改
2、懒加载，使用的时候才更新缓存
使用的时候才从DB中加载
也可以采用异步的方式填充缓存
开启一个线程 定时将DB的数据刷到缓存中
高并发脏读的三种情况
1、先更新数据库，再更新缓存
update与commit之间，更新缓存，commit失败
则DB与缓存数据不一致
2、先删除缓存，再更新数据库
update与commit之间，有新的读，缓存空，读DB数据到缓存 数据是旧的数据
commit后 DB为新数据
则DB与缓存数据不一致
3、先更新数据库，再删除缓存（推荐）
update与commit之间，有新的读，缓存空，读DB数据到缓存 数据是旧的数据
commit后 DB为新数据
则DB与缓存数据不一致
采用延时双删策略
----------------------------------------------
2:Read/Write Through Pattern
应用程序只操作缓存，缓存操作数据库。
Read-Through（穿透读模式/直读模式）：应用程序读缓存，缓存没有，由缓存回源到数据库，并写入
缓存。（guavacache） Write-Through（穿透写模式/直写模式）：应用程序写缓存，缓存写数据库。
该种模式需要提供数据库的handler，开发较为复杂。
----------------------------------------------
3:Write Behind Caching Pattern
应用程序只更新缓存。
缓存通过异步的方式将数据批量或合并后更新到DB中
不能时时同步，甚至会丢数据
```

#### 3:缓存架构的设计思路

```
缓存的整体设计思路包括：
1、多层次 :分布式缓存宕机，本地缓存还可以使用
2、数据类型:
简单数据类型
Value是字符串或整数或二进制
Value的值比较大（大于100K）
只进行setter和getter
可采用Memcached
Memcached纯内存缓存，多线程 K-V
复杂数据类型
Value是hash、set、list、zset
需要存储关系，聚合，计算
可采用Redis
3:3、要做集群
分布式缓存集群方案（Redis）
codis
哨兵+主从
RedisCluster
4、缓存的数据结构设计
1、与数据库表一致
数据库表和缓存是一一对应的
缓存的字段会比数据库表少一些
缓存的数据是经常访问的
用户表，商品表
```

#### 4: redis 事务

所谓事物 是指作为单个逻辑工作单元执行的一系列操作

```
Atomicity（原子性）：构成事务的的所有操作必须是一个逻辑单元，要么全部执行，要么全部不执行 ,Redis:一个队列中的命令 执行或不执行
Consistency（一致性）：数据库在事务执行前后状态都必须是稳定的或者是一致的。 Redis: 集群中不能保证时时的一致性，只能是最终一致性
Isolation（隔离性）：事务之间不会相互影响。 Redis: 命令是顺序执行的，在一个事务中，有可能被执行其他客户端的命令的
Durability（持久性）：事务执行成功后必须全部写入磁盘。 Redis有持久化但不保证 数据的完整性

Redis的事务是通过multi、exec、discard和watch这四个命令来完成的。
Redis的单个命令都是原子性的，所以这里需要确保事务性的对象是命令集合。
Redis将命令集合序列化并确保处于同一事务的命令集合连续且不被打断的执行
Redis不支持回滚操作
```

####  5: 事务命令

```
multi：用于标记事务块的开始,Redis会将后续的命令逐个放入队列中，然后使用exec原子化地执行这个
命令队列
exec：执行命令队列
discard：清除命令队列
watch：监视key
unwatch：清除监视key
```

如下图所示

![](/img/redis2info1.png)

```
127.0.0.1:6379> multi OK
127.0.0.1:6379> set s1 222 QUEUED 
127.0.0.1:6379> hset set1 name zhangfei QUEUED 
127.0.0.1:6379> exec 
1) OK
2) (integer) 1 
127.0.0.1:6379> multi OK
127.0.0.1:6379> set s2 333 QUEUED 
127.0.0.1:6379> hset set2 age 23 QUEUED 
127.0.0.1:6379> discard OK
127.0.0.1:6379> exec (error) ERR EXEC without MULTI 
127.0.0.1:6379> watch s1 OK
127.0.0.1:6379> multi OK
127.0.0.1:6379> set s1 555 QUEUED 
127.0.0.1:6379> exec # 此时在没有exec之前，通过另一个命令窗口对监控的s1字段进行 修改(nil) 
127.0.0.1:6379> get s1 222 
127.0.0.1:6379> unwatch 
OK
```

#### 6:事务机制

```
事务的执行
1. 事务开始
在RedisClient中，有属性flags，用来表示是否在事务中
flags=REDIS_MULTI
2. 命令入队
RedisClient将命令存放在事务队列中
（EXEC,DISCARD,WATCH,MULTI除外）
3. 事务队列
multiCmd *commands 用于存放命令
4. 执行事务
RedisClient向服务器端发送exec命令，RedisServer会遍历事务队列,执行队列中的命令,最后将执
行的结果一次性返回给客户端。
如果某条命令在入队过程中发生错误，redisClient将flags置为REDIS_DIRTY_EXEC，EXEC命令将会失败
返回。
```

如下图所示
![](/img/redis2info2.png)


```
Watch的执行
使用WATCH命令监视数据库键
redisDb有一个watched_keys字典,key是某个被监视的数据的key,值是一个链表.记录了所有监视这个数
据的客户端。
监视机制的触发
当修改数据后，监视这个数据的客户端的flags置为REDIS_DIRTY_CAS
事务执行
RedisClient向服务器端发送exec命令，服务器判断RedisClient的flags，如果为REDIS_DIRTY_CAS，则
清空事务队列。
```

如下图所示


![](/img/redis2info3.png)
#### 7: redis 的弱事务性

```
Redis语法错误
整个事务的命令在队列里都清除
Redis运行错误
在队列里正确的命令可以执行 （弱事务性）
弱事务性 ： 
1、在队列里正确的命令可以执行 （非原子操作）
2、不支持回滚
Redis不支持事务回滚（为什么呢）
大多数事务失败是因为语法错误或者类型错误，这两种错误，在开发阶段都是可以预见的
Redis为了性能方面就忽略了事务回滚。 （回滚记录历史版本）
```

