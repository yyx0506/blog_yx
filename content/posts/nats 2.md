
---
title: "nats streaming 安装部署 "
date: 2022-08-22
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


### nats 2

#### 1:Nats streaming是什么

```
NATS Streaming是一个以NATS为驱动的数据流系统且它的源码也是由Golang语言编写的。其中NATS Streaming服务是一个可执行的文件名为：nats-streaming-server。NATS Streaming与底层NATS服务平台无缝嵌入、扩展和互动。NATS Streaming服务作为开源软件在MIT许可下载，Apcera也积极的在维护和支持NATS Streaming 服务。

功能 : 除了NATS平台核心的特性外,NATS Streaming提供了以下一些功能特性
增强消息协议:NATS Streaming使用谷歌协议缓冲区实现自己的增强型消息格式。这些消息通过二进制数据流在NATS核心平台进行传播,因此不需要改变NATS的基本协议。NATS Streaming信息包含以下字段:
    　　序列 - 一个全局顺序序列号为主题的通道
    　　主题 - 是NATS Streaming 交付对象
    　　答复内容 - 对应"reply-to"对应的对象内容
    　　数据 - 真是数据内容
    　　时间戳 - 接收的时间戳，单位是纳秒
    　　重复发送 - 标志这条数据是否需要服务再次发送
    　　CRC32 - 一个循环冗余数据校验选项，在数据存储和数据通讯领域里，为了保证数据的正确性所采用的检错手段，这里使用的是 IEEE CRC32 算法
消息/事件的持久性:NATS Streaming提供了可配置的消息持久化，持久目的地可以为内存或者文件。另外，对应的存储子系统使用了一个公共接口允许我们开发自己自定义实现来持久化对应的消息
至少一次的发送:NATS Streaming提供了发布者和服务器之间的消息确认(发布操作) 和订阅者和服务器之间的消息确认(确认消息发送)。其中消息被保存在服务器端内存或者辅助存储(或其他外部存储器)用来为需要重新接受消息的订阅者进行重发消息。
发布者发送速率限定:NATS Streaming提供了一个连接选项叫 MaxPubAcksInFlight，它能有效的限制一个发布者可能随意的在任何时候发送的未被确认的消息。当达到这个配置的最大数量时，异步发送调用接口将会被阻塞，直到未确认消息降到指定数量之下。
每个订阅者的速率匹配／限制:NATS Streaming运行指定的订阅中设置一个参数为 MaxInFlight，它用来指定已确认但未消费的最大数据量，当达到这个限制时，NATS Streaming 将暂停发送消息给订阅者，直到未确认的数据量小于设定的量为止
以主题重发的历史数据:新订阅的可以在已经存储起来的订阅的主题频道指定起始位置消息流。通过使用这个选项,消息就可以开始发送传递了:
    　　1. 订阅的主题存储的最早的信息
    　　2. 与当前订阅主题之前的最近存储的数据，这通常被认为是 "最后的值" 或 "初值" 对应的缓存
    　　3. 一个以纳秒为基准的 日期／时间
    　　4. 一个历史的起始位置相对当前服务的 日期／时间，例如：最后30秒
    　　5. 一个特定的消息序列号
 持久订阅：订阅也可以指定一个“持久化的名称”可以在客户端重启时不受影响。持久订阅会使得对应服务跟踪客户端最后确认消息的序列号和持久名称。当这个客户端重启或者重新订阅的时候，使用相同的客户端ID 和 持久化的名称，对应的服务将会从最早的未被确认的消息处恢复
```

#### 2: nats streaming 的安装部署

```
之前已经部署过nats 这次就把 nats 和nats streaming 一起合到一起
version: '3'
networks:
  nats-net:
    external: true
services:
  nats1:
    image: nats:2.1.9
    ports:
      - 4222:4222
      - 8222:8222
    networks:
      - nats-net
    command: -m 8222 --user "nats" --pass "P@ssw0rd" --cluster nats://0.0.0.0:6222 --routes nats://nats3:6222
  nats2:
    image: nats:2.1.9
    ports:
      - 4223:4222
      - 8223:8222
    networks:
      - nats-net
    command: -m 8222 --user "nats" --pass "P@ssw0rd" --cluster nats://0.0.0.0:6222 --routes nats://nats1:6222
  nats3:
    image: nats:2.1.9
    ports:
      - 4224:4222
      - 8224:8222
    networks:
      - nats-net
    command: -m 8222 --user "nats" --pass "P@ssw0rd" --cluster nats://0.0.0.0:6222 --routes nats://nats2:6222

  nats-streaming1:
    image: nats-streaming:0.20.0
    depends_on:
      - nats1
      - nats2
      - nats3
    networks:
      - nats-net
    volumes:
      - /var/lib/nats/nats-streaming1/data:/nats/data
      - /var/log/nats/nats-streaming1/log:/nats/log
    command: "--store file --dir /nats/data -clustered --cluster_log_path /nats/log --cluster_id nats-cluster --cluster_node_id nats-streaming1  --cluster_peers nats-streaming2,nats-streaming3 --cluster_sync --nats_server nats://nats:P@ssw0rd@nats1:4222,nats://nats:P@ssw0rd@nats2:4222,nats://nats:P@ssw0rd@nats3:4222"

  nats-streaming2:
    image: nats-streaming:0.20.0
    depends_on:
      - nats1
      - nats2
      - nats3
    networks:
      - nats-net
    volumes:
      - /var/lib/nats/nats-streaming2/data:/nats/data
      - /var/log/nats/nats-streaming2/log:/nats/log
    command: "--store file --dir /nats/data -clustered --cluster_log_path /nats/log --cluster_id nats-cluster --cluster_node_id nats-streaming2 --cluster_peers nats-streaming1,nats-streaming3 --cluster_sync --nats_server nats://nats:P@ssw0rd@nats1:4222,nats://nats:P@ssw0rd@nats2:4222,nats://nats:P@ssw0rd@nats3:4222"

  nats-streaming3:
    image: nats-streaming:0.20.0
    depends_on:
      - nats1
      - nats2
      - nats3
    networks:
      - nats-net
    volumes:
      - /var/lib/nats/nats-streaming3/data:/nats/data
      - /var/log/nats/nats-streaming3/log:/nats/log
    command: "--store file --dir /nats/data -clustered --cluster_log_path /nats/log --cluster_id nats-cluster --cluster_node_id nats-streaming3 --cluster_peers nats-streaming1,nats-streaming2 --cluster_sync --nats_server nats://nats:P@ssw0rd@nats1:4222,nats://nats:P@ssw0rd@nats2:4222,nats://nats:P@ssw0rd@nats3:4222"
    
    

其中 4222 接口 为开放 客户端连接 8222 接口为观察开放端  该特点为
1 通过nats + nats-streaming 搭建3节点nats集群，nats提供服务;
2 支持认证;
3 nats-streaming 提供节点和消息持久化;
```

#### 3 ： 生产与消费

```
### 直接选择sc 连接 生产者

import asyncio
import time
import msgpack
from common.service.nats_service import NatsService

class NatsProducer(object):

    sc = None

    @classmethod
    async def test(cls):
        cls.sc = await NatsService.get_stan()
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

###消费者 
import traceback
import asyncio
import msgpack
from common.service.nats_service import NatsService

class NatsConsumer(object):

    subject = 'l:test:index_data'

    @classmethod
    async def start(cls):
        sc = await NatsService.get_stan()
        await sc.subscribe(cls.subject,  cb=cls.msg_handler)
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



