---
title: "Grpc"
date: 2023-07-21
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


### Grpc

#### 1 grpc 是什么

gRPC（gRPC Remote Procedure Calls）是一种现代的开源远程过程调用（RPC）框架，由Google开发并于2015年发布。它基于HTTP/2协议进行通信，用于在不同的计算机之间进行高效、可靠和跨平台的远程调用。

gRPC 使用Protocol Buffers（简称ProtoBuf或protobuf）作为默认的序列化机制，这是一种轻量且高效的数据交换格式，可以用于定义数据结构和接口。gRPC支持多种编程语言，包括但不限于C++, Java, Python, Go, Node.js, Ruby等，这使得不同语言的应用程序可以通过gRPC进行通信。

与传统的REST API相比，gRPC有以下一些特点：

1. 高性能：gRPC使用HTTP/2协议进行通信，可以复用连接、支持多路复用和二进制传输，从而减少延迟并提高性能。
2. 强类型接口：gRPC使用Protocol Buffers来定义接口和数据类型，使得通信双方在编译时就能知道可用的方法和参数，从而减少出错概率。
3. 双向流支持：gRPC支持双向流式通信，客户端和服务器可以同时发送和接收数据流，适用于实时应用场景。
4. 自动代码生成：根据ProtoBuf定义文件，gRPC可以自动生成客户端和服务器端的代码，简化开发过程。
5. 支持拦截器：gRPC允许开发人员实现拦截器，对请求和响应进行处理，例如认证、日志记录等。

由于其高性能、强类型接口和跨语言支持，gRPC在微服务架构和分布式系统中得到了广泛应用。它逐渐成为许多大型互联网公司构建可靠和高效服务的首选框架。

官方网站 ：https://grpc.io/

中文官方文档：https://doc.oschina.net/grpc

#### 2: 安装PROTOBUF

```
https://github.com/protocolbuffers/protobuf/releases
打开链接 然后选择对应的版本以及系统
然后下一步解压完放在一个目录下  将此目录复制粘贴至系统变量的PATH 下 即可 CMD 运行 protoc 即可查看是否配置成功
D:\protoc-23.4-win64\bin
然后安装   注意的是老版本的是 GITHUB 开头的  现在最新的都是GOOGLE 开头的
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest

我们 声明一个PROTO 文件
syntax = "proto3";

option go_package = "./service";

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloResponse) {}
}

message HelloRequest {
  string requestName = 1;
}

message HelloResponse {
  string ResponseMsg = 1;
}


下一步是生成PB文件  一个是生成普通的PB文件一个是生成GRPC的文件
protoc --go_out=. *.proto
protoc --go-grpc_out=. *.proto

下面是这二者的区别 
protoc --go_out=. 和 protoc --go-grpc_out=. 都是用于使用 Protocol Buffers 编译器（protoc）生成 Go 语言代码的选项，但它们有不同的功能：

protoc --go_out=.:
这个选项用于生成纯粹的 Protocol Buffers 的 Go 语言代码。也就是说，它只会生成消息结构体和与消息相关的辅助代码，但不会生成 gRPC 相关的代码。gRPC 是一个基于 Protocol Buffers 的远程过程调用（RPC）框架，可以用于构建分布式系统。

使用这个选项时，protoc 会根据定义在 .proto 文件中的消息类型和服务描述生成对应的 Go 结构体和方法。生成的代码将包含序列化和反序列化等消息处理功能，但不包含与 gRPC 相关的内容。

protoc --go-grpc_out=.:
这个选项用于生成与 gRPC 相关的 Go 语言代码。它会生成 gRPC 的服务接口和客户端存根代码，以便您可以基于 gRPC 框架构建分布式应用程序。

当您的 .proto 文件中定义了 gRPC 服务，并且想要在 Go 中实现这些服务时，使用 --go-grpc_out 选项会生成相应的 gRPC 服务接口和实现，以及客户端存根代码。

综上所述，区别在于一个选项仅生成 Protocol Buffers 的 Go 代码，而另一个选项除了 Protocol Buffers 的 Go 代码外，还会生成与 gRPC 相关的代码，如 gRPC 服务接口和客户端存根。通常情况下，如果您使用了 gRPC 服务，那么需要同时使用这两个选项来生成完整的 Go 代码。

```

#### 3：服务端和客户端代码的编写

服务端编写：

创建grpc server 对象，可以理解为Server端的抽象对象

将Server (需要被调用的服务端接口) 注册到 grpc  server的内部注册中心

创建listen监听TCP端口

grpc server 开始 lis.Accept 直到STOP

```
package main

import (
	"log"
	"net"

	"golang.org/x/net/context"
	// 导入grpc包
	"google.golang.org/grpc"
	// 导入刚才我们生成的代码所在的proto包。
	pb "go_preject/grpc/proto/service"
	"google.golang.org/grpc/reflection"
)

// 定义server，用来实现proto文件，里面实现的Greeter服务里面的接口
type server struct {
	pb.UnimplementedGreeterServer
}

// 实现SayHello接口
// 第一个参数是上下文参数，所有接口默认都要必填
// 第二个参数是我们定义的HelloRequest消息
// 返回值是我们定义的HelloReply消息，error返回值也是必须的。
func (s *server) SayHello(ctx context.Context, in *pb.HelloRequest) (*pb.HelloResponse, error) {
	// 创建一个HelloReply消息，设置Message字段，然后直接返回。
	return &pb.HelloResponse{ResponseMsg: "Hello " + in.RequestName}, nil
}

func main() {
	// 监听127.0.0.1:50051地址
	lis, err := net.Listen("tcp", "127.0.0.1:50051")
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}

	// 实例化grpc服务端
	s := grpc.NewServer()

	// 注册Greeter服务
	pb.RegisterGreeterServer(s, &server{})

	// 往grpc服务端注册反射服务
	reflection.Register(s)

	// 启动grpc服务
	if err := s.Serve(lis); err != nil {
		log.Fatalf("failed to serve: %v", err)
	}
}

```



客户端编写

创建与给定目标（服务端）的连接交互

创建server的客户端对象

发送RPC请求，等待同步响应，得到回调后返回响应结果

输出响应结果

```
package main

import (
	"golang.org/x/net/context"
	"log"
	"time"
	// 导入grpc包
	"google.golang.org/grpc"
	// 导入刚才我们生成的代码所在的proto包。
	pb "go_preject/grpc/proto/service"
)

const (
	defaultName = "world"
)

func main() {
	// 连接grpc服务器
	conn, err := grpc.Dial("localhost:50051", grpc.WithInsecure())
	if err != nil {
		log.Fatalf("did not connect: %v", err)
	}
	// 延迟关闭连接
	defer conn.Close()

	// 初始化Greeter服务客户端
	c := pb.NewGreeterClient(conn)

	// 初始化上下文，设置请求超时时间为1秒
	ctx, cancel := context.WithTimeout(context.Background(), time.Second)
	// 延迟关闭请求会话
	defer cancel()

	// 调用SayHello接口，发送一条消息
	r, err := c.SayHello(ctx, &pb.HelloRequest{RequestName: "world"})
	if err != nil {
		log.Fatalf("could not greet: %v", err)
	}

	// 打印服务的返回的消息
	log.Printf("Greeting: %s", r.ResponseMsg)
}

```

