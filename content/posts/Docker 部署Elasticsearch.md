---
title: "Docker 部署Elasticsearch"
date: 2021-05-10
categories:
- tranquilpeak
- features
tags:
- docker
- elasticsearch
# keywords:
# - elasticsearch
# - docker
thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->



### Docker 部署Elasticsearch

本篇文章主要介绍的是docker安装，所以常规安装就不赘述了。

```
docker search elasticsearch 查看相关版本
docker pull elasticsearch 如果不加入tag 会默认下载最新版本
docker pull elasticsearch:7.6.2 这里我下载的是7.6.2版本的
docker run -d --name elasticsearch  -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.6.2 然后启动该镜像

```

#### 参数说明

```
--name表示镜像启动后的容器名称  

-d: 后台运行容器，并返回容器ID；

-e: 指定容器内的环境变量

-p: 指定端口映射，格式为：主机(宿主)端口:容器端口 最后跟着的是启动容器对应的镜像
```

#### 直接访问9200端口 如图所示即为成功
![](/img/es1.png)

#### 使用elasticsearch–head可视化查看es

![](/img/es2.png)

#### 然后下一步 是 进入到此elasticsearch 容器内部 

```
docker exec -it 容器id  /bin/bash
cd config 
vi elasticsearch.yml 增加下面两列参数后保存退出
http.cors.enabled: true
http.cors.allow-origin: "*"
然后 exit 退出
最后重启此容器 docker restart 容器id 这样就可以跨域请求了
```



最后给一个小的tips https://chrome.google.com/webstore/detail/elasticsearch-csv-exporte/kjkjddcjojneaeeppobfolgojhohbpjn es kibana 谷歌浏览器工具
https://chrome.google.com/webstore/category/extensions 谷歌应用商店的地址


