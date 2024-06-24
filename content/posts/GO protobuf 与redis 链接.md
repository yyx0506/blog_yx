
---
title: "GO protobuf 与redis 链接"
date: 2022-10-24
categories:
- tranquilpeak
- features
tags:
- golang
# keywords:
# - golang
thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->

### GO protobuf 与redis 链接

https://golang.google.cn/   go  官方链接 

#### 1：GO  生成 protobuf 对应的go 文件

```
作为go 语言如何使用 protobuf呢 ？
其实跟python 一样 首先第一件事 就是将protobuf 文件转化为 go文件 （python 是转化为 py文件）

protoc --go_out=plugins=grpc:. *.proto

有一点要说明的是 执行此命令时  proto 文件需要 加入 指定的go_packege 才可以

syntax = "proto3";
option go_package="./protobuf_go";
package author;

message AuthorBase{
    string      author_id                   = 1;           //id
    string      name                        = 2;           //名字
}

总是写命令 那么我们写一个 shell 脚本 


#!/bin/sh
SRC_DIR=./
DST_DIR=./

#GO
mkdir -p $DST_DIR

for f in $(find $SRC_DIR -iname "*.proto"); do
    protoc --go_out=$DST_DIR/ $f
done


每次执行只需要 sh gen.sh 就可以了
```

#### 2： Go 使用protobuf 

```
package main

import (
    "log"
    "fmt"
    // 辅助库
    "github.com/golang/protobuf/proto"

    // AuthorBase.pb.go 的路径
    "pb/protobuf_go"
)



// 将数据通过路径下的 protobuf go 文件  使用辅助函数设置域的值
ip_data := &protobuf_go.AuthorBase{
Id: int64(domain_id),
Ip: avc[0].String(),
}

// 进行编码
datas, err := proto.Marshal(ip_data)

// 进行解码
newTest := &protobuf_go.AuthorBase{}
err = proto.Unmarshal(data, newTest)
if err != nil {
log.Fatal("unmarshaling error: ", err)
}


```

#### 3：GO 链接redis

```
import "github.com/go-redis/redis"

首先导入redis 链接库


var swg sync.WaitGroup

var RedisClient1 *redis.Client = NewClient("0.0.0.0", "6001", "123145464", 6)


func NewClient(host string, port string, password string, db int) *redis.Client {

	RedisClient := redis.NewClient(&redis.Options{
		Addr:     host + ":" + port, // use default Addr
		Password: password,          // no password set
		DB:       db,                // use default DB
	})
	return RedisClient
}

// 从 test list 中 取数据
result, err := RedisClient1.LPop("test").Result()
		if err != nil {
			fmt.Println("blpop error")
		} else {
			fmt.Println(result)
		}

```

