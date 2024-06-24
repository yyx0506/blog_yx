---
title: "kafka python 链接"
date: 2022-08-09
categories:
- tranquilpeak
- features
tags:
- kafka
# keywords:
# - kafka

thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->


### kafka4

#### 1 python 异步 Kafka 链接

```
我们主要的用到的库是aiokafka
下面是用单例模式写的一个 python 连接 注意的是kafka 不同于redis mysql 等 producer 和consumer 是分开的


import asyncio
import collections
from apps.app import AppCtx
from common.config.kafka_config import *
from aiokafka import AIOKafkaConsumer,AIOKafkaProducer

class KafkaService():
    instances = {}

    kafka_config_production = {
        'test': {"host": PRO_KAFKA_HOST, "port": PRO_KAFKA_PORT}
    }
    kafka_config_development = {
        'test': {"host": DEVE_KAFKA_HOST, "port": DEVE_KAFKA_PORT}
    }

    @classmethod
    async def get_producer(cls,kafka_name='test'):
        instance = cls.instances.get(kafka_name)
        if instance is None:
            loop = asyncio.get_event_loop()
            config = cls.kafka_config_production.get(
                kafka_name) if AppCtx.get_env() == 'production' else cls.kafka_config_development.get(kafka_name)
            address = '%s:%s' % (config['host'], config['port'])
            consumer = AIOKafkaProducer(loop=loop, bootstrap_servers=address)
            await consumer.start()
            cls.instances[kafka_name] = instance = consumer
        return instance

    @classmethod
    async def get_consumer(cls, kafka_name='test',topic='my_topic'):
        instance = cls.instances.get(kafka_name)
        if instance is None:
            loop = asyncio.get_event_loop()
            config = cls.kafka_config_production.get(
                kafka_name) if AppCtx.get_env() == 'production' else cls.kafka_config_development.get(kafka_name)
            address = '%s:%s' % (config['host'], config['port'])
            consumer = AIOKafkaConsumer(topic,loop=loop, bootstrap_servers=address,group_id='aaaa', auto_offset_reset='earliest')
            await consumer.start()
            cls.instances[kafka_name] = instance = consumer
        return instance



```

#### 2: kafka producer

```
import asyncio
from common.service.kafka_service import KafkaService


class KafkaProducer(object):

    @classmethod
    async def producer_send(cls):
        kafka_cli = await KafkaService().get_producer()
        await kafka_cli.send(
        "my_topic",
        key="Each+ Apollo".encode(),
        value="is nice".encode())
        await kafka_cli.stop()


if __name__ == '__main__':
    asyncio.run(KafkaProducer().producer_send())



```

#### 3 Kafka consumer

```
 import asyncio
from common.service.kafka_service import KafkaService

class KafkaConsumer(object):
    #topic：主题，需要和生产者的topic保持一致，不然收不到正确消息
    # group_id：消费者的组别，可以是任意的，但是不要重复
    # auto_offset_reset：收消息的模式，“earliest”, “latest”,默认是后者
    # loop: 协程循环事件
    # bootstrap_servers：ip端口连接池
    # metadata_max_age_ms：强制刷新事件，单位毫秒
    @classmethod
    async def producer_consumer(cls):
        kafka_cli = await KafkaService().get_consumer()
        while 1:
            data = await kafka_cli.getmany()
            print(data)
            if data:
                for tp, message in data.items():
                    print('tp>>>', tp)
                    print(tp.topic)
                    print('message>>', message)
                    for msg in message:
                        print(msg)
                        print(msg.timestamp, msg.offset)
                        v = msg.value
                        print(v)
                ### 断开
        await kafka_cli.stop()

if __name__ == '__main__':
    asyncio.run(KafkaConsumer().producer_consumer())
```

#### tips: 以上就是aiokafka 的简答代码 后续会继续介绍Kafka 的高阶操作

