---
title: "redis 介绍"
date: 2022-06-03
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

### redis 介绍1

#### 1: 获取数据生产

```
keys：用于获取当前数据库的模式匹配的所有key

smembers：获取set集合中的所有元素
```

当获取一个set集合中的数据超过100w 时不建议用 smembers 而是要使用 sscan 进行迭代 

```
而scan又包含多个类似命令

SCAN 增量迭代当前数据库中的数据库键。
SSCAN 增量迭代集合键中的元素。
HSCAN 增量迭代哈希键中的键值对。
ZSCAN 增量迭代有序集合中的元素（包括元素成员和元素分值）。
也就是说，keys、smembers和scan家族命令的最大区别是：

          keys和smembers是获取全部，如果当redis中key的数量过去庞大（或者set的元素很多），则很耗费内存，会阻塞redis几秒钟

         scan家族是逐步增量获取。即遍历获取一定数量的key或者元素，在获取一定数量的key或元素，不会一次获取。

那么，scan命令就比keys、smembers命令好吗？不是这样的，scan命令家族也是有缺点的。由于scan采用的增量迭代，当redis中的key是随时变化的，比如key增加减少或者key的名字变更，这种情况，scan就暴露他的弊端了，可能无法获取所有的key了。

所以采用哪种方式获取key或者获取元素，得根据自己的业务，如果你key是随时变化，就采用keys或者smembers吧。因为我们业务中redis的初始化只是在项目启动时初始化一次，所以在获取set的全部元素时采用的sscan命令。
```

Example:

```
import asyncio
from common.service.aioredis_service import AioRedisService


async def do():
redis = await AioRedisService.get('diandian')

cur = b'0'
while cur:
cur, values = await redis.sscan('set:test', cur, count=10000)
print(cur, values)
return

```

#### 2: redis 单线程

```
在一开始的时候，Redis采用的是单线程模型，因为Redis是一个基于内存的数据库，将所有的数据放入内存，所以使用单线程的操作效率是最高的，多线程会上下文切换消耗大量时间，对于内存系统来说，单线程才能产生更高的效率。但是单线程不意味着整个Redis就一个线程，Redis其他模块还有各自的线程
使用I/O多路复用机制同时监听多个客户端，在单线程模式下，即使网络处理很多，因为存在I/O多路复用，依然可以在高速的内存处理下得到忽略。
这个I/O threads 指的是网络I/O处理方面使用了多线程，如网络数据的读写和协议解析等，这是因为读写网络的read/write在Redis执行期间占用了大部分CPU时间，如果把这部分模块使用多线程方式实现会对性能带来极大地提升。但是Redis执行命令的核心模块还是单线程的。
需要注意的是在 Redis6.0 中，多线程机制默认是关闭的，需要在 redis.conf 中完成以下两个设置才能启用多线程。
```



