

---
title: "nats 2.0"
date: 2023-02-04
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

### nats 2.0

#### 1关于nats

```
nats 目前是有3个产品 其中 第二个 已经被弃用了 第三个是最新nats 开源的持久化消息队列

core-nats: 不做持久化的及时信息传输系统
nats-streaming: 基于 nats 的持久化消息队列(已弃用)
nats-jetstream: 基于 nats 的持久化消息队列



```

#### 2 nats-jetstream

```
现在由于nats-streaming 已被弃用 所以我们 讲一下 nats-jetstream
NATS JetStream是NATS生态系统的下一代流系统，为NATS 2.0打造，具有分布式安全、多租户和水平扩展能力。

官方地址 https://github.com/nats-io/nats.py

jetstream 提供持久化 nats 服务, 客户端支持实时推送的 push 模式和自定义拉取的 pull 模式
subject: 和 nats 一样, 用来区分不同的消息
stream: 定义了消息的储存方式, 保留规则, 丢弃规则 (stream 和 subject 是 1:n 的关系)
consumer: 定义了消息接受方式并记录接受到的位置, 有 2 种消费方式及时推送 push 和自定义拉取 pull (consumer 和 stream 是 1:n 的关系)

```

#### 3 python如何使用  nats-jetstream

```
一下是官方给出的示例

import asyncio
import nats
from nats.errors import TimeoutError

async def main():
    nc = await nats.connect("localhost")

    # Create JetStream context.
    js = nc.jetstream()

    # Persist messages on 'foo's subject.
    await js.add_stream(name="sample-stream", subjects=["foo"])

    for i in range(0, 10):
        ack = await js.publish("foo", f"hello world: {i}".encode())
        print(ack)

    # Create pull based consumer on 'foo'.
    psub = await js.pull_subscribe("foo", "psub")

    # Fetch and ack messagess from consumer.
    for i in range(0, 10):
        msgs = await psub.fetch(1)
        for msg in msgs:
            await msg.ack()
            print(msg)

    # Create single ephemeral push based subscriber.
    sub = await js.subscribe("foo")
    msg = await sub.next_msg()
    await msg.ack()

    # Create single push based subscriber that is durable across restarts.
    sub = await js.subscribe("foo", durable="myapp")
    msg = await sub.next_msg()
    await msg.ack()

    # Create deliver group that will be have load balanced messages.
    async def qsub_a(msg):
        print("QSUB A:", msg)
        await msg.ack()

    async def qsub_b(msg):
        print("QSUB B:", msg)
        await msg.ack()
    await js.subscribe("foo", "workers", cb=qsub_a)
    await js.subscribe("foo", "workers", cb=qsub_b)

    for i in range(0, 10):
        ack = await js.publish("foo", f"hello world: {i}".encode())
        print("\t", ack)

    # Create ordered consumer with flow control and heartbeats
    # that auto resumes on failures.
    osub = await js.subscribe("foo", ordered_consumer=True)
    data = bytearray()

    while True:
        try:
            msg = await osub.next_msg()
            data.extend(msg.data)
        except TimeoutError:
            break
    print("All data in stream:", len(data))

    await nc.close()

if __name__ == '__main__':
    asyncio.run(main())
    
    

pip3 install nats-py


import nats

class NatsService():
    instances = {}
    instances_stan = {}
    config_production = {

    }
    config_development = {

    }

    @classmethod
    async def get(cls, name='test'):
        instance = cls.instances.get(name)
        if instance is None:
            nc = await nats.connect(cls.config_production[name])
            js = nc.jetstream()
            cls.instances[name] = instance = js
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


这是一个单例的链接模式

下面是推送端的处理：
nats_cli = await NatsService.get(server_type)
await nats_cli.publish(topic, data)
```

