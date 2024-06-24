
---
title: "nats 介绍 搭建"
date: 2022-08-20
categories:
- tranquilpeak
- features
tags:
- nats
# keywords:
# - nats

thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->

### nats1

#### 1:nats的介绍

```
nats 时go语言开发的开源的轻量高性能的云原生消息系统
nats消息由主题处理，不依赖于网络位置。它提供了应用程序或服务与底层物理网络之间的抽象层。
```

#### 2:nats 和string 的搭建

```
我们直接选docker-compose 进行部署
GitHub 有大佬已经把配置文件写好了，我们直接拉一份即可  地址 :https://github.com/xiliangMa/nats-cluster
git clone https://github.com/xiliangMa/nats-cluster.git
cd docker-compose/
docker network create nats-net
docker-compose -f nats&nats-streaming.yaml up -d
```

#### 3 nats 搭建

```
docker-compose.yaml 文件配置

version: "3.5"
services:
   nats-server:
       #image: provide/nats-server:latest
       image: nats:latest
       volumes:
           - ./nats-server.conf:/nats-server.conf
       ports:
           - 4222:4222
       restart: always
 
nats-server.conf 文件配置 

 # Client port of 4222 on all interfaces
port: 4222

# HTTP monitoring port
monitor_port: 8222

# This is for clustering multiple servers together.
cluster {

  # Route connections to be received on any interface on port 6222
  port: 6222

  # Routes are protected, so need to use them with --routes flag
  # e.g. --routes=nats-route://ruser:T0pS3cr3t@otherdockerhost:6222
  authorization {
    user: nats
    password: nats123
    timeout: 2
  }

  # Routes are actively solicited and connected to from this server.
  # This Docker image has none by default, but you can pass a
  # flag to the gnatsd docker image to create one to an existing server.
  routes = []
}

```

 #### 4:nats链接书写

```
同样是单例模式的书写
import time
import random
import string
from apps.app import AppCtx
from common.config.nats_config import *
from nats.aio.client import Client as NATS
from stan.aio.client import Client as STAN


class NatsService():
    instances = {}
    instances_stan = {}
    config_production = {
        'test': {"servers": PRO_NATS_SERVERS, "user": PRO_NATS_USER, "password": PRO_NATS_PASSWORD}
    }
    config_development = {
        'test': {"servers": DEVE_NATS_SERVERS, "user": DEVE_NATS_USER, "password": DEVE_NATS_PASSWORD}
    }

    @classmethod
    async def get(cls, name='test'):
        instance = cls.instances.get(name)
        if instance is None:
            config = cls.config_production[name] if AppCtx.get_env() == 'production' else cls.config_development[
                name]
            nc = NATS()
            cls.instances[name] = instance = nc
            config['max_reconnect_attempts'] = 1800
            await nc.connect(**config)
        return instance

    @classmethod
    async def get_stan(cls, name='test'):
        instance = cls.instances_stan.get(name)
        if instance is None:
            config = cls.config_production[name] if AppCtx.get_env() == 'production' else cls.config_development[
                name]
            nc = NATS()
            config['max_reconnect_attempts'] = 1800
            await nc.connect(**config)
            sc = STAN()
            cls.instances_stan[name] = instance = sc
            client_id = '%s%s' % (random.choice(
                string.ascii_uppercase), int(time.time() * 1000))
            await sc.connect('cluster', client_id, nats=nc)
        return instance

    @classmethod
    async def close(cls):
        '''关闭连接
        '''
        instances = cls.instances
        for i in instances:
            if instances[i]:
                await instances[i].close()
        cls.instances = {}
        instances_stan = cls.instances_stan
        for i in instances_stan:
            if instances_stan[i]:
                await instances_stan[i].close()
        cls.instances_stan = {}

```

#### 5:nats producer

```

import asyncio
import time
import msgpack
from common.service.nats_service import NatsService

class NatsProducer(object):

    sc = None
    @classmethod
    async def test(cls):
        cls.sc = await NatsService.get()
        while 1:
            change_data = {
                'time': int(time.time()),
                'type': 1,
                'sub_type': 3,  # 1 更新  2 新增  3删除
                'ids': [1,2,3,4,5,6]}
            print(change_data)
            await cls.sc.publish('l:test:index_data', msgpack.packb(change_data))
            await asyncio.sleep(1)

if __name__ == '__main__':
    asyncio.run(NatsProducer().test())
```

#### 6: nats consumer

```
import traceback
import asyncio
import msgpack
from common.service.nats_service import NatsService

class NatsConsumer(object):

    subject = 'l:test:index_data'

    @classmethod
    async def start(cls):
        nc = await NatsService.get()
        await nc.subscribe(cls.subject, cls.subject, cb=cls.msg_handler)
        while True:
            await asyncio.sleep(3)

    @classmethod
    async def msg_handler(cls, msg):

        try:
            if msg.data:
                data = msgpack.unpackb(msg.data, raw=False)
                print(data)
        except Exception as e:
            print(e)
            traceback.print_exc()

if __name__ == '__main__':
    asyncio.run(NatsConsumer().start())
```

#### 7:nats 中的 nats.aio.client stan.aio.client 有什么区别

```
stan.aio.client 就是nats 的集群 nats-Streaming 
nats.aio.client 是普通的单机服务 所以当大量的使用时 用 nath-streaming
Start session with NATS Streaming cluster 
```

