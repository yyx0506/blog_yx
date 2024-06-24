---
title: "redis 问题总结1"
date: 2022-07-20
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

### redis 问题总结1

#### 1:Redis 的 RDB 和 AOF 机制各自的优缺点是什么？这两种机制是否可以混合使用？

```
RDB 优点：采用⼦进程生成 RDB ⽂件，减少对 Redis 主线程的阻塞，保证 Redis 性能；RDB 是内存快照，恢复效率⾼。
RDB 不⾜：RDB 是间隔⼀段时间⽣成⼀次，两次 RDB 创建之前，如果Redis 发⽣故障，会发⽣数据丢失。
AOF 优点：采用 always 配置项时，可以每个命令做⼀次持久化，可靠性相比 RDB 更⾼。
AOF 不足：AOF ⽂件较⼤，会触发 AOF 重写，重写时会竞争内存资源；AOF 恢复是回放命令操作，恢复速度慢于 RDB。
Redis 4.0 后，提供了混合持久化功能，RDB 以⼀定的频率执⾏，在两次快照之间，使用 AOF 日志记录这期间的所有命令操作，避免了⼀直记录 AOF 的开销（例如 AOF 重写）。
```

#### 2: Redis 经常被称为单线程的系统，你如何理解 Redis 的单线程模型？

```
Redis6.0 前的请求解析、键值数据读写、结果返回都是由⼀个线程完成，所以称 Redis 为单线程模型。
Redis 单线程容易阻塞，为了避免阻塞，Redis 设计了⼦进程和异步线程的⽅式来完成某些耗时操作，例如使用⼦进程实现 RDB ⽣成，AOF 重写；使用异步线程实现数据删除等。
Redis 6.0 开始，使用多个线程来完成请求解析和结果返回，提升对⽹络请求的处理，提升系统整体的吞吐量。
Redis 6.0 对于键值数据的读写仍然使用单线程来处理。
单线程的优势是开发简单、可以保证命令执⾏的原⼦性；不⾜是容易被阻塞。
```

#### 3: Redis 的事务操作有哪些命令？Redis 事务能保证原子性吗？

```
事务操作基本命令包括 MULTI、EXEC、DISCARD 和 WATCH。MULTI 表示事务开始；EXEC用来实际执⾏事务中命令；DISCARD 表示放弃事务执⾏；WATCH 本质是⼀个乐观锁，用于监控⼀个或多个键，如果被监控的键中有⼀个被修改或删除了，那么事务就放弃执⾏。
Redis 事务的原⼦性保证建议分三种情况解释：
	 命令都正常执⾏，此时原⼦性可以保证；
	 命令⼊队时出错，EXEC 时会拒绝执⾏所有命令，原⼦性可以保证；
   命令实际执⾏时出错，Redis 会执⾏剩余命令，原⼦性得不到保证。
所以，Redis 事务不保证原⼦性。
```

