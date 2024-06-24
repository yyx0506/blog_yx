
---
title: "k8s 使用"
date: 2022-10-17
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


### K8s 的使用

#### 1: docker 转 k8s

```
大部分人最开始使用的是docker 我也不例外 那么之前部署好的docker-compose 怎么转换成 k8s呢

方式有很多，今天只介绍一种 通过kompose 

休闲是下载 kompose

# Linux
curl -L https://github.com/kubernetes/kompose/releases/download/v1.26.0/kompose-linux-amd64 -o kompose

# macOS
curl -L https://github.com/kubernetes/kompose/releases/download/v1.26.0/kompose-darwin-amd64 -o kompose

# Windows
curl -L https://github.com/kubernetes/kompose/releases/download/v1.26.0/kompose-windows-amd64.exe -o kompose.exe

#  给权限 并将目录移到 bin 目录下
chmod +x kompose
sudo mv ./kompose /usr/local/bin/kompose

你只需要一个现有的 docker-compose.yml 文件 下一步进入到 此文件的同级目录下 
执行  kompose convert 即可
最后可以得到 类似于 *-deployment.yaml  *-service.yaml  的文件
```

#### 2: 安装k8s

```
网上有很多 搭建的方式 想简单一点可以直接使用 kubesphere 或者rancher 这种第三方平台提供的工具

阿里云服务器 需要执行 

cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

yum install  kubectl kubelet kubeadm -y (下载最新版本的  )

然后 需要 设置一下配置文件
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> /etc/profile
source /etc/profile

启动 服务 
systemctl enable kubelet.service  


下一步 设置主机解析
# 每台主机上都需要需要进行配置




使用 kubesphere 安装一套
官方文档地址 https://kubesphere.io/zh/docs/v3.3/quick-start/all-in-one-on-linux/

执行一下命令

export KKZONE=cn

curl -sfL https://get-kk.kubesphere.io | VERSION=v2.3.0 sh -

chmod +x kk

./kk create cluster --with-kubernetes v1.22.12 --with-kubesphere v3.3.1


最后验证结果即可 
kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l 'app in (ks-install, ks-installer)' -o jsonpath='{.items[0].metadata.name}') -f

可以使用默认的帐户和密码 (admin/P@88w0rd) 通过 <NodeIP>:30880 访问控制台。

```

