---
title: "Docker-compose 部署ELK"
date: 2021-05-11
categories:
- tranquilpeak
- features
tags:
- docker
- elasticsearch
- kibana
- logstash
# keywords:
# - docker
# - elasticsearch
# - kibana
# - logstash
# - docker-compose
thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->


### Docker-compose 部署ELK

```
第一步切换到/usr/local目录下
#创建docker目录
mkdir docker
mkdir -p ./elasticsearch/data  # 创建Elasticsearch数据挂载路径
chmod 777 ./elasticsearch/data  # 解决启动elasticsearch容器出现root权限问题
mkdir -p ./elasticsearch/plugins  #创建Elasticsearch插件挂载路径
mkdir -p ./logstash # 创建Logstash配置文件存储路径
在该路径下创建logstash-febs.conf配置文件:避免直接运行logstash出现异常
vim ./logstash/logstash.conf
然后进入到此文件夹后面 创建一个 docker-compose.yml 文件 具体配置如下

input {
  tcp {
    mode => "server"
    host => "0.0.0.0"
    port => 4560
    codec => json_lines
  }
}
output {
  elasticsearch {
    hosts => "es:9200"
    index => "boutique-logstash-%{+YYYY.MM.dd}"
    #user => elastic #用户名
    #password => "nana123456"   #es 配置了密码 把这个打开
  }
}

在 elasticsearch config 文件夹下 创建elasticsearch.yml
network.host: 0.0.0.0  #使用的网络
http.cors.enabled: true #跨域配置
http.cors.allow-origin: "*"

```

```


version: '3.1'
services:
  elasticsearch:
    image: elasticsearch:7.6.2
    container_name: elk_elasticsearch
    restart: always  #开机启动，失败也会一直重启
    environment:
      - "cluster.name=elk_elasticsearch" #集群名称为elk_elasticsearch
      - "discovery.type=single-node" #单节点启动
      - "ES_JAVA_OPTS=-Xms256m -Xmx256m" #jvm内存分配为512MB
    volumes:
      - ./elasticsearch/plugins:/usr/share/elasticsearch/plugins
      - ./elasticsearch/data:/usr/share/elasticsearch/data
      - ./elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml #配置文件挂载
    ports:
      - 9200:9200
  kibana:
    image: kibana:7.6.2
    container_name: elk_kibana
    restart: always  #开机启动，失败也会一直重启
    links:
      - elasticsearch:es #配置elasticsearch域名为es
    depends_on:
      - elasticsearch  #kibana在elasticsearch启动之后再启动
    environment:
      - "elasticsearch.hosts=http://es:9200" #因为上面配置了域名，所以这里可以简写为http://es:9200
    ports:
      - 5601:5601
  logstash:
    image: logstash:7.6.2
    container_name: elk_logstash
    restart: always  #开机启动，失败也会一直重启
    volumes:
      - ./logstash/logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    depends_on:
      - elasticsearch #logstash在elasticsearch启动之后再启动
    links:
      - elasticsearch:es #可以用es这个域名访问elasticsearch服务
    ports:
      - 4560:4560
```

```
然后直接执行 docker-compose up -d 后台启动。 可以通过es head 进行查看
```
![](/img/elkinstall1.png)



#### kibana 换成中文

```\
#进入容器 kibana 所在的容器
docker exec -it [容器编号|容器名称] bash
cd config
vi kibana.yml
i18n.locale: "zh-CN"

```

