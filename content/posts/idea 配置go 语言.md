---
title: "idea 配置go 语言"
date: 2023-02-22
categories:
- tranquilpeak
- features
tags:
- go
# keywords:
# - k8s

thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->


### idea 配置go 语言

#### 1：安装go 

```
首先查看 现有电脑上的go 环境 which go
如果没有 则此步骤略 如果有 直接 sudo rm -rf 路径名 一般是在  /usr/local/go 目录下

下一步 https://golang.google.cn/dl/ 去官方网站下载 go 确认好自己下载的版本

直接傻瓜式一路安装即可。go默认会安装在/usr/local/go文件夹下

然后编译 vim ~/.bash_profile export GO_HOME=/usr/local/go 即可

最后测试 go version 出现对应的版本信息即可


```

#### 2：idea 安装对应的插件

```
在 plugins 中搜索 go 然后安装 下载后重新idea生效，配置goroot

goroot 指的是go sdk安装路径;  gopath指的是go依赖存放的地方 类似于 java中的maven 一般来说现在都是用gomod去取代 gopath了


```

#### 3：可能会出现的问题

```
 IDEA 选不了sdk，The selected directory is not a valid home for Go Sdk
选中/usr/local/go目录后报错如下：
The selected directory is not a valid home for Go Sdk
解决: 修改/usr/local/go/src/runtime/internal/sys/zversion.go，在文件中添加版本号
sudo  vim /usr/local/go/src/runtime/internal/sys/zversion.go
最后一行添加如下: 一定要和你下载的sdk版本一致。
const TheVersion = 'go1.19.6'
```

#### 4: 下载依赖

```
将自己的go项目 从 仓库拉下来 后 
直接执行下面命令 即可下载 依赖 
go mod tidy
```

#### 5: 打包镜像

```
FROM golang:go1.19.6
WORKDIR /data/test/
COPY . .

RUN apk add --no-cache make gcc musl-dev git ca-certificates \
    && go env -w GO111MODULE=on \
    && go env -w GOPROXY=https://goproxy.io,direct \
    && go get \
    && go build main.go \
    && chmod a+x main

#EXPOSE 80

这样的我们的项目就直接打包在镜像里面了
```



