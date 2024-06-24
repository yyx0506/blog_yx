---
title: "Nginx介绍"
date: 2022-04-28
categories:
- tranquilpeak
- features
tags:
- nginx
# keywords:
# - nginx

thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->


### nginx 介绍

nginx 是一个**高性能的HTTP和反向代理web服务器**，**同时也提供了IMAP/POP3/SMTP服务,**特点是**占有内存少，并发能力强**

Nginx提供的负载均衡策略有2种：内置策略和扩展策略。内置策略为轮询，加权轮询，Ip hash。扩展策略，就天马行空，只有你想不到的没有他做不到的**

![](/img/nginx1.png)
#### 加权轮询，因为可能不同服务器性能不一样，能接收的请求数量不同
![](/img/nginx2.png)

第三个就是nginx可以做动静分离

nginx 默认监听的是80端口

nginx 常用命令

```
cd /usr/local/nginx/sbin/
./nginx  启动
./nginx -s stop  停止
./nginx -s quit  安全退出
./nginx -s reload  重新加载配置文件
ps aux|grep nginx  查看nginx进程

```

nginx 相关配置书写 第一块为全局配置 可以全局生效制定用户进程

```
user  nginx;

events {
    worker_connections   1000;
} # 监听的事件 最大连接数

http {
# upstream 流配置 管理负载均衡
    ... 
	upstream cvzhanshi{
		server 127.0.0.1:8082/ weight=1;
		server 127.0.0.1:8081/ weight=1;
	}

# 80 默认http的请求 443 默认 https的请求
    server {
        listen       80;
        server_name  localhost;
        # 根部路 listen 的端口会走到此目录下
        location / {
            root   html;
            index  index.html index.htm;
	     proxy_pass http://cvzhanshi; # 代理路径
        }
        ...
}

```

