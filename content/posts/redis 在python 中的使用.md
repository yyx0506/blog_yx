---
title: "redis 在python 中的使用 "
date: 2022-06-15
categories:
- tranquilpeak
- features
tags:
- redis
# keywords:
# - redis
# - python

thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->

### redis 在python 中的使用 

####   1:string 

```
------------------------------------------基础操作------------------------------------------
单个 key的设置  await cls.redis_cli.set(key,value)    单个 key 的时间复杂度都是O(1) 多个都是 O(n)
单个 key的获取  await cls.redis_cli.get(key)
单个 key的删除  await cls.redis_cli.delete(key)+
多个 key的设置  await cls.redis_cli.mset(*key_value_list) key_value_list 是一个列表 第一位是key 第二位是value 以此类推
多个 key的获取  await cls.redis_cli.mget(*key_list) key_list 是一个列表 里面是要获取的key
多个 key的删除  await cls.redis_cli.delete(*key_list)  同上
------------------------------------------高阶操作------------------------------------------
获取单个 key 的value的长度 await cls.redis_cli.strlen(key) 
单个key 对value的追加 await cls.redis_cli.append(key,value) 如果key 不存在就 相当于set 如果存在就在原先的值后追加value 
单个key 的 incr await cls.redis_cli.incr(key) 如果key 不存在就直接设置0 ，存在就累加1
单个key 的 decr await cls.redis_cli.decr(key) 如果key 不存在就直接设置0 ，存在就减去1 
单个key 的 setnx  await cls.redis_cli.setnx(key,value)  如果key 不存在 就设置value 如果存在就不操作
多个key 的 setnx  await cls.redis_cli.msetnx(*key_value_list)    key_value_list 同上
是一个原子性(atomic)操作， 所有给定键要么就全部都被设置， 要么就全部都不设置， 不可能出现第三种状态
对 string key 的 游标操作  count 的值 最好在10000以下
cur = b'0'
while cur:
  	cur, keys = await redis.scan(cur, 'key:*', count=10000)
  	if not keys:
	  		break

```

#### 2:hash 

```
------------------------------------------基础操作------------------------------------------
单个 hash key 设置 await cls.redis_cli.hset(key,field,value)
单个 hash key 获取 await cls.redis_cli.hget(key,field) 
单个 hash key 删除 一个field await cls.redis_cli.hdel(key,field) 
单个 hash key 删除 多个field await cls.redis_cli.hdel(key,*field)
多个 hash key 设置 await cls.redis_cli.hmset(*key_list,*field_value_list) field_value_list 第一位是field第二位value
多个 hash key 获取 await cls.redis.hmget(key, *fields) 
多个 hash key 删除 多个field await cls.redis_cli.hdel(key,*field) 
------------------------------------------高阶操作------------------------------------------
获取单个 field 的value的长度 await cls.redis_cli.hstrlen(key,field)
单个field 的 setnx  await cls.redis_cli.hsetnx(key,field,value)  如果field 不存在 就设置value 如果存在就不操作
获取hash 表中的所有域 await cls.redis_cli.hkeys(key)
获取hash 表中的所有域的值 await cls.redis_cli.hvals(key)
获取hash 表中的所有 域和值 await cls.redis_cli.hgetall(key)
cur = b'0'
    while cur:
    cur, keys = await redis.hscan('hash:*', cursor=0, count=1)
    print(keys)

```

#### 3:list

```
------------------------------------------基础操作------------------------------------------
单个 value 加入到 list await cls.redis_cli.lpush(key,value)  -- lpush 从左边插入 rpush 从右边插入
单个 value 加入到 list await cls.redis_cli.lpushx(key,value)  --如果key 不存在就什么也不做 lpushx 从左边插入 rpushx 从右边插入
单个 value 从list 取出 await cls.redis_cli.lpop(key)  -- lpop 从左边插入 rpop 从右边插入
多个 value 加入到 list list await cls.redis_cli.lpush(key,*value)  -- lpush 从左边插入 rpush 从右边插入
查看列表的长度 await cls.redis_cli.llen(key)
获取 列表中下标的元素  await cls.redis.lindex(key, 0) -1 表示最后一个元素
删除 列表 await cls.redis.del(key)
------------------------------------------高阶操作------------------------------------------
获取列表内的全部元素  await cls.redis_cli.lrange( key, 0, -1 ) 

```

#### 4:set

```
------------------------------------------基础操作------------------------------------------
单个 value 加入到 set await cls.redis_cli.sadd(key,value)  
判断 单个value 是否在 集合中 await cls.redis_cli.sismember(key,value)
移除并返回集合中的一个随机元素 await cls.redis_cli.spop(key)
返回集合中元素 并不删除 await cls.redis_cli.srandmember(key,count) count 不带默认是1
移除在集合中的元素 await cls.redis_cli.srem(key,value)
获取集合中元素的个素 await cls.redis_cli.scard(key)
------------------------------------------高阶操作------------------------------------------
获取集合中的元素 await cls.redis_cli.smembers(key) 建议在 50w 以下的集合中使用
cur = b'0'
while cur:
    cur, words = await data_redis.sscan(key, cursor=cur, count=10000)
    if not words:
				print(cur,words)
```

#### 5: zset

```
------------------------------------------基础操作------------------------------------------
单个 value 加入到 zset await cls.redis_cli.zadd(key,value,score) score 值可以是整数值或双精度浮点数
多个 value 加入到 zset await cls.redis_cli.zadd(key,*value_score_list) 第一位是value 第二位是score 以此类推
获取 zset 中 score 值在某个范围内的数量 await cls.redis.zcount(key.score1,score2)
获取 zset 中 key 的 value数 await cls.redis_cli.zcard(key)

------------------------------------------高阶操作------------------------------------------
 cur = b'0'
 count = 1000
 while cur:
   cur, values = await cls.redis_cli.zscan(key, cur, count=count)
   if not values:
   		continue
   else:
   		print(values)
```





