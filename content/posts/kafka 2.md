---
title: "kafka 机制"
date: 2022-08-07
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

### kafka 2

#### 1:Kafka零拷⻉机制保证读数据⾼性能

```
消费者读取数据流程：
      1. 消费者发送请求给kafka服务
      2.kafka服务去os cache缓存读取数据（缓存没有就去磁盘读取数据）
      3.从磁盘读取了数据到os cache缓存中
      4.os cache复制数据到kafka应⽤程序中
      5.kafka将数据（复制）发送到socket cache中
      6.socket cache通过⽹卡传输给消费者
如图所示 ------
```
![](/img/kafka1.png)


```
kafka linux sendfile技术 — 零拷⻉
      1. 消费者发送请求给kafka服务
      2.kafka服务去os cache缓存读取数据（缓存没有就去磁盘读取数据）
      3.从磁盘读取了数据到os cache缓存中
      4.os cache直接将数据发送给⽹卡
      5.通过⽹卡将数据传输给消费者
如图所示--------
```
![](/img/kafka2.png)


#### 2:Kafka⽇志分段保存

```
Kafka中⼀个主题，⼀般会设置分区；
⽐如创建了⼀个topic_a，然后创建的时候指定了这个主题有三个分区。
其实在三台服务器上，会创建三个⽬录。
服务器1（kafka1）：
    创建⽬录topic_a-0:
        ⽬录下⾯是我们⽂件（存储数据），kafka数据就是message，数据存储在log⽂件⾥.log结尾的就是⽇志⽂件，在kafka中把数据⽂件就叫            				做⽇志⽂件。
        ⼀个分区下⾯默认有n多个⽇志⽂件（分段存储），⼀个⽇志⽂件默认1G
服务器2（kafka2）：
		创建⽬录topic_a-1:
服务器3（kafka3）：
		创建⽬录topic_a-2:
如图所示------
```
![](/img/kafka3.png)


#### 3 Kafka 二分法查找定位数据

```
Kafka⾥⾯每⼀条消息，都有⾃⼰的offset（相对偏移量），存在物理磁盘上⾯，在position
Position：物理位置（磁盘上⾯哪个地⽅）
也就是说⼀条消息就有两个位置：
      offset：相对偏移量（相对位置）
      position：磁盘物理位置
稀疏索引：
			Kafka中采⽤了稀疏索引的⽅式读取索引，kafka每当写⼊了4k⼤⼩的⽇志（.log），就往
index⾥写⼊⼀个记录索引。
			其中会采⽤⼆分查找
如图所示------
```

![](/img/kafka4.png)