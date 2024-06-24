---
title: "redis 数据迁移 "
date: 2022-11-19
categories:
- tranquilpeak
- features
tags:
- redis
# keywords:
# - redis
# - k8s

thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->

### redis 数据迁移

#### 1 准备一台新的redis

```
最近线上一台自建redis服务的服务器频繁报警，内存使用率有点高，这是一台配置比较低的机器，所以再考虑数据迁移的问题 
于是查阅了一些资料，最后决定使用 阿里自研的RedisShake工具


第一步先准备一个 新的 redis 这里直接使用kubesphere 部署一个redis 

kubectl get namespace 首先 查看一下命名空间 我之前是设置了一个命名空间这次就直接用这个

然后从docker-hub 上拉取redis 之后

首先编辑一下 redis-sts.yaml
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
  namespace: work-space
  labels:
    app: redis
spec:
  serviceName: redis-headless
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
        - name: redis
          image: 'docker.io/redis:latest'
          command:
            - "redis-server"
          args:
            - "/etc/redis/redis.conf"
          ports:
            - name: redis-6379
              containerPort: 6379
              protocol: TCP
          volumeMounts:
            - name: config
              mountPath: /etc/redis
            - name: data
              mountPath: /data
          resources:
            limits:
              cpu: '1'
              memory: 4000Mi
            requests:
              cpu: 100m
              memory: 500Mi
      volumes:
        - name: config
          configMap:
            name: redis-config
            items:
              - key: redis-config
                path: redis.conf
  volumeClaimTemplates:
    - metadata:
        name: data
        namespace: zdevops
      spec:
        accessModes: [ "ReadWriteOnce" ]
        storageClassName: "local"
        resources:
          requests:
            storage: 5Gi

---
apiVersion: v1
kind: Service
metadata:
  name: redis-headless
  namespace: work-space
  labels:
    app: redis
spec:
  ports:
    - name: redis-6379
      protocol: TCP
      port: 6379
      targetPort: 6379
  selector:
    app: redis
  clusterIP: None
  type: ClusterIP


然后编辑一下 redis-cm.yaml
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: redis-config
  namespace: work-space
data:
  redis-config: |
    appendonly yes
    protected-mode no
    dir /data
    port 6379
    requirepass nana123456
    
storageclass.storage.k8s.io "glusterfs" not found 
接下来 我们 直接运行 kubectl apply -f . 创建资源 

有一个问题 是 harbor 的镜像似乎不能和 k8s 的镜像拉取 这个还要后续再研究  然后 目前部署的不能够外部访问 
```

#### 2 设置redis 的外部访问

```
Kubesphere 生成的 redis 刚开始是无法 外部访问 需要设置

第一步选择服务 冉闯创新新服务 命名为 外部访问服务后 我们直接选择  指定工作负载 选择 刚才定义好的 redis 服务
最后选择 外部访问 定义完端口就可以了

```

#### 3 使用RedisShake 做redis 迁移

```
正式操作前先在测试环境实践一把看看效果如何，先说明下环境
源库：192.168.1.xxx
目标库：192.168.1.xxx

1 : 先下载 
wget https://github.com/alibaba/RedisShake/releases/download/v3.0.0/redis-shake-v3.0.0.tar.gz
然后 直接解压 
版本 可以 自己去git 进行选择
2 :解压 
tar -zxvf redis-shake-v3.0.0.tar.gz
mv redis-shake-linux-amd64 /usr/local/bin/redis-shake
3: 更改配置文件redis-shake.toml
主要是更改source.address source.password  target.address target.password
4: 开始脚本执行
redis-shake ./redis-shake.toml
```

