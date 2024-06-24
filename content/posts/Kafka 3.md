---
title: "kafka 搭建"
date: 2022-08-08
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

### Kafka 3

#### 1: 使用docker-compose 搭建kafka

```
首先要有docker 以及docker-compose 的环境 最好是高版本的
1: 创建目标目录
mkdir -p /opt/zookeeper && mkdir -p /opt/kafka
2: 创建docker-compose.yml 文件
version: '3.3'
services:
  zookeeper:
    container_name: zookeeper
    image: wurstmeister/zookeeper
    ports:
      - "2181:2181"
    restart: always
  kafka:
    container_name: kafka
    image: wurstmeister/kafka
    depends_on: [ zookeeper ]
    ports:
      - "9092:9092"
    environment:
      KAFKA_ADVERTISED_HOST_NAME: 10.0.4.13
      KAFKA_CREATE_TOPICS: "test:1:1"
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
    volumes:
      - /etc/localtime:/etc/localtime
      - /data/product/zj_bigdata/data/kafka/docker.sock:/var/run/docker.sock
    restart: always

3 启动docker-compose 
docker-compose up -d

```

#### 2: 使用docker-compose 搭建 kafkaui

```
kafkaui 就类似一个网页可以去管理观察kafka 类似于 navicate 之于mysql 
1: 创建目标目录
mkdir -p /opt/kafkaui
2: 创建docker-compose.yml 文件
version: '3.4'
services:
  kafka-ui:
    image: provectuslabs/kafka-ui:master
    container_name: kafka-ui
    ports:
      - 8080:8080
    restart: always
    environment:
      - KAFKA_CLUSTERS_0_NAME=douyh
      - KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS=10.0.4.13:9092
      # - KAFKA_CLUSTERS_0_PROPERTIES_SECURITY_PROTOCOL=SASL_PLAINTEXT
      # - KAFKA_CLUSTERS_0_PROPERTIES_SASL_MECHANISM=PLAIN
      # - KAFKA_CLUSTERS_0_PROPERTIES_SASL_JAAS_CONFIG=org.apache.kafka.common.security.plain.PlainLoginModule required username="app" password="xxxxxxxx";
      # - KAFKA_CLUSTERS_0_PROPERTIES_PROTOCOL=SASL
      - AUTH_TYPE="LOGIN_FORM"
      - SPRING_SECURITY_USER_NAME=yyx
      - SPRING_SECURITY_USER_PASSWORD=yyx123


3 启动docker-compose 
docker-compose up -d
在网页端直接 http://121.5.58.115:8080/ui/clusters/douyh/all-topics?perPage=25 即可看到效果
```

#### 3: python 连接kafka

```
pip3 install kafka-python

---------------------------------producer 代码 ---------------------------------------
from kafka import KafkaProducer
producer = KafkaProducer(bootstrap_servers=['121.5.58.115:9092'])
msg1 = "zxc"
producer.send(topic='test', value=msg1.encode(),partition=0,key='456'.encode())
producer.close()
-----------------------------------consumer 代码 --------------------------------------
from kafka import KafkaConsumer
consumer = KafkaConsumer('test', bootstrap_servers=['121.5.58.115:9092'])
for msg in consumer:
    recv = "%s:%d:%d: key=%s value=%s" % (msg.topic, msg.partition, msg.offset, msg.key, msg.value)
    print(recv)

```



