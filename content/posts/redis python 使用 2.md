---
title: "redis python 使用 2"
date: 2023-03-17
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

### redis python 使用 2

#### 1：redis-py

```
之前一直使用的是aioredis 这个库 后面发现这个库已经不更新了
https://github.com/aio-libs/aioredis-py
作者的意思是他会将此代码接入到 redis-py 的asyncio 分支中 所以以后我们直接使用redis-py 即可 这也是官方提供的Python连接redis的库
所以我们直接安装redis-py的最新版本
然后按照aioredis的说法重新去写一个单例模式  比如
from apps.app import AppCtx
from redis import asyncio as aioredis

class AsRedisService(object):

    instances = {'k_redis_n':{}}

    redis_config_production = {
       
    }

    redis_config_development = {
        'text': {"host": "0.0.0.0", "port": "6739", "password": "11111111"}, 
    }

    @classmethod
    async def get(cls, redis_name='k_redis_n', db=0):
        if cls.instances[redis_name].get(db) is None:
            config = cls.redis_config_production.get(
                redis_name) if AppCtx.get_env() == 'production' else cls.redis_config_development.get(redis_name)
            redis_cluster = AppCtx.get_redis_cluster().get(redis_name, '')
            if redis_cluster and redis_cluster in config:
                config = config.get(redis_cluster, {})
            address = 'redis://%s:%s' % (config['host'], config['port'])
            cls.instances[redis_name][db] = await aioredis.from_url(address, db=db, password=config.get('password'), socket_keepalive=True, health_check_interval=10)
        return cls.instances[redis_name][db]

```

#### 2 一些特点

```
Zadd变动 变成zadd 字典
例如这样 字典中的 key 代表 value  value 代表score

async def test1(cls):
  redis_cli = await AsRedisService().get('text',3)
  for i in range(20):
    dict_ii = {i:int(time.time())}
    await redis_cli.zadd('test_time',dict_ii)
    await asyncio.sleep(1)



hmset变动 去除hmset_dict变成hmset ，原先的hmset变成了一定要设置字典
mset 变成了字典 {k:v}

这两个变动跟zadd 类似 

下面这个变动有一点大

pipe变动 async with redis.pipeline(transaction=False) as pipe:
encoding 去除

@classmethod
async def test2(cls):
  redis_cli = await AsRedisService().get('text',3)
  async with redis_cli.pipeline(transaction=False) as pipe:
    pipe.set('text2', 'sssss')
    pipe.lpush('text3','sssaa')
    await pipe.execute()

多个语句可以放到一个管道里面同时执行
```

