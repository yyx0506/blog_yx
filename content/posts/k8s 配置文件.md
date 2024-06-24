
---
title: "k8s 配置文件"
date: 2022-11-11
categories:
- tranquilpeak
- features
tags:
- k8s
# keywords:
# - k8s

thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->

### k8s 配置文件

#### 1:命名空间

```
1 ： 编写 yaml 文件 
# 之前 最开始部署都是利用 kompose 这个工具 将 docker 的配置文件转化为 k8s 的配置文件 其实就是yaml文件 

我们先编写一个简单的 yaml 文件  
首先就是yaml 的基本语法
yaml 文件的书写 就是要 注意空格 他并不在意空格的多少 只要相同层级的元素左侧对齐即可
使用 # 进行注释
vim  namespace.yaml
---
apiversion:  v1     #  api版本
kind:  Namespace    #  创建类型----（Namespace命名空间)
metadata：            #  元数据
    name:  ns-monitor    #  起个名字   （给元数据的名字）
    labels:                   #  标签
        name:  ns-monitor       #  标签的名字

2: 创建资源
kubectl  apply  -f  namespace.yaml
# 没有问题时 会显示 ******* created
kubectl  apply  -f  namespace.yaml  --validate  （出现问题时 查看报错的信息）

3 ： 查看所有命名空间里的资源
 kubectl  get  namespace
 
4: 查看命名空间内的某个资源

kubectl  get  namespace  ns-monitor

5 删除命名空间
kubectl  delete  -f  namespace.yaml
kubectl  delete  namespace  ns-monitor
```

#### 2: pod

```
1:编写pod 文件
vim  pod.yaml

---
apiversion:  v1
kind:  Pod
metadata:
    name:  website 
    labels:
        app:  website      #  打标签
spec:                       #  指定的意思（属性，定义的是Pod）
    containers：       #   定义容器
         -  name:  test-website         #  容器的名字，自定义
             image：  daocloud.io/librery/nginx         #  镜像
             ports:
                 -  containerPort:  80            #  容器暴露的端口
nodeName:  k8s-node1                       #  指定node节点的名称

2: 查看pod
#    kubectl  delete  pod  --all   //批量删除
#    kubectl  get  pods

3: 查看pod运行在那台机器上
kubectl  get  pods   -o  wide

4:  查看pod定义的详细信息
kubectl  get  pod  website  -o  yaml

5: 查看kubectl describe 支持查询Pod的状态和生命周期事件
kubectl  desribe  pod  website


各字段含义：

spec：指定资源的内容
Name: Pod的名称
Namespace: Pod的Namespace。
Image(s): Pod使用的镜像
Node: Pod所在的Node。
Start Time: Pod的起始时间
Labels: Pod的Label。
Status: Pod的状态。
Reason: Pod处于当前状态的原因。
Message: Pod处于当前状态的信息。
IP: Pod的PodIP
Replication Controllers: Pod对应的Replication Controller。
===============================
2.Containers:Pod中容器的信息
Container ID: 容器的ID
Image: 容器的镜像
Image ID:镜像的ID
State: 容器的状态
Ready: 容器的准备状况(true表示准备就绪)。
Restart Count: 容器的重启次数统计
Environment Variables: 容器的环境变量
Conditions: Pod的条件，包含Pod准备状况(true表示准备就绪)
Volumes: Pod的数据卷
Events: 与Pod相关的事件列表
=====
生命周期：指的是status通过# kubectl get pod
生命周期包括：running、Pending、completed、

```

#### 3： 实战测试

```
1: 创建 资源申请 k8s 中 内存空间等资源的申请文件
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  creationTimestamp: null
  labels:
    io.kompose.service: signature-claim0
  name: signature-claim0
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
status: {}


2: 创建资源信息  Deployment 作为一个 K8s 资源，它有自己的 metadata 元信息
首先要有一个核心的字段，即 replicas，这里定义期望的 Pod 数量为1个

apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    kompose.cmd: kompose convert
    kompose.version: 1.26.0 (40646f47)
  creationTimestamp: null
  labels:
    io.kompose.service: signature
  name: signature
spec:
  replicas: 1
  selector:
    matchLabels:
      io.kompose.service: signature
  strategy:
    type: Recreate
  template:
    metadata:
      annotations:
        kompose.cmd: kompose convert
        kompose.version: 1.26.0 (40646f47)
      creationTimestamp: null
      labels:
        io.kompose.service: signature
    spec:
      containers:
        - args:
            - node
            - ./search_keyword_sign.js
          image: cr.nana1024.com/yanyuxiang/dy_searchkeyword_signature:1.0.0
          name: signature
          ports:
            - containerPort: 9931
          resources: {}
          volumeMounts:
            - mountPath: /etc/localtime
              name: signature-claim0
      restartPolicy: Always
      volumes:
        - name: signature-claim0
          persistentVolumeClaim:
            claimName: signature-claim0
status: {}



3:创建pod yaml

apiVersion: v1
kind: Service
metadata:
  annotations:
    kompose.cmd: kompose convert
    kompose.version: 1.26.0 (40646f47)
  creationTimestamp: null
  labels:
    io.kompose.service: signature
  name: signature
spec:
  ports:
    - name: "9931"
      port: 9931
      targetPort: 9931
  selector:
    io.kompose.service: signature
status:
  loadBalancer: {}
  
  
  
4:  kubectl  apply  -f . 
一起执行 一个 k8s 的pod 就创建完毕了
```



