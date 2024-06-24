

---
title: "kafka 介绍"
date: 2022-08-06
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


### Kafka  1

之前一直有用过nats 这次想看看 kafka 就研究了一下

#### 1: 消息系统的特点

```
1、解耦合
2、异步处理
例如电商平台，秒杀活动。
        ⼀般流程会分为：
        		1：⻛险控制、2：库存锁定、3：⽣成订单、4：短信通知、5：更新数据
        通过消息系统将秒杀活动业务拆分开，将不急需处理的业务放在后⾯慢慢处理；
        流程改为：
					1：⻛险控制、2：库存锁定、3:消息系统、4:⽣成订单、5：短信通知、6：更新数据
3、流量的控制
        3.1 ⽹关在接受到请求后，就把请求放⼊到消息队列⾥⾯
        3.2 后端的服务从消息队列⾥⾯获取到请求，完成后续的秒杀处理流程。然后再给⽤户返回结果。
        优点：控制了流量
        缺点：会让流程变慢
```



#### 2 Kafka 核心概念

```
⽣产者：Producer 往Kafka集群⽣成数据
消费者：Consumer 往Kafka⾥⾯去获取数据，处理数据、消费数据
Kafka的数据是由消费者⾃⼰去拉去Kafka⾥⾯的数据
主题：topic
分区：partition
默认⼀个topic有⼀个分区（partition），⾃⼰可设置多个分区（分区分散存储在服务器不同
节点上）
解决了⼀个海量数据如何存储的问题
例如：有2T的数据，⼀台服务器有1T，⼀个topic可以分多个区，分别存储在多台服
务器上，解决海量数据存储问题
```

#### 3 Kafka的集群架构

```
Kafka集群中，⼀个kafka服务器就是⼀个broker
Topic只是逻辑上的概念，partition在磁盘上就体现为⼀个⽬录
Consumer Group：消费组
消费数据的时候，都必须指定⼀个group id，指定⼀个组的id
假定程序A和程序B指定的group id号⼀样，那么两个程序就属于同⼀个消费组
特殊：
⽐如，有⼀个主题topicA
程序A去消费了这个topicA，那么程序B就不能再去消费topicA（程序A和程序B属于
⼀个消费组）
再⽐如程序A已经消费了topicA⾥⾯的数据，现在还是重新再次消费topicA的数据，
是不可以的，但是重新指定⼀个group id号以后，可以消费。
不同消费组之间没有影响。消费组需⾃定义，消费者名称程序⾃动⽣成（独⼀⽆⼆）。
Controller：Kafka节点⾥⾯的⼀个主节点。借助zookeeper  所以kafka 是需要借助 zookeeper 的
```

#### 4: kafka磁盘顺序写保证写数据的性能

```
kafka写数据：顺序写，往磁盘上写数据时就是追加数据，没有随机写的操作
经验：如果一个服务器磁盘达到一定的个数，磁盘也达到一定的转数往磁盘⾥⾯顺序写（追加写）数据的速度和写内存的速度差不多
⽣产者⽣产消息，经过kafka服务先写到os cache 内存中，然后经过sync顺序写到磁盘上
```



