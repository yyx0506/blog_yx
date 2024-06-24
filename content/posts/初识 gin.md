
---
title: "初识 gin"
date: 2023-05-09
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

### 初识 gin

#### 1:gin 是什么

```
Gin是一个golang的微框架，封装比较优雅，API友好, 源代码比较明确。 具有快速灵活，容错方便等特点。其实对于golang而言，web框架的依赖远比Python，Java之类的要小。自身的net/http足够简单，性能也非常不错。框架更像是一个常用函数或者工具的集合。借助框架开发，不仅可以省去很多常用的封装带来的时间，也有助于团队的编码风格和形成规范。

gin 官方网址 ： https://gin-gonic.com/
github 地址：   https://github.com/gin-gonic/gin

Gin特点和特性

速度： Gin之所以被很多企业和团队使用，第一个原因是因为速度快，性能表现出众
中间件: 和iris类似, gin在处理请求时,支持中间件操作, 方便编码处理
路由: gin中可以非常简单的实现路由解析功能，并包含路由组解析功能
内置渲染: Gin支持JSON、XML和HTML等多种数据格式的渲染， 并提供了方便的操作API


官方文档很好用: https://gin-gonic.com/docs/


```

#### 2:直接上手使用gin框架

```
环境配置 ： mac 环境
ida: idea 2021.3 （选择这个是因为多种语言都可以兼顾）
go version : go version go1.19.6 

第一步我们新建一个项目 gin_demo 然后在此文件下新建一个go mod 文件 (类似于java中的maven)

下一步安装gin  所有go 的安装包使用的都是 go get 命令
go get -u github.com/gin-gonic/gin

在下一步 GoModules 中的 enable 一定要勾选中
1、Preferences -> Plugins 搜索 Go、File Watchers 两个扩展进行安装
2、Preferences -> Languages & Frameworks -> Go 配置GoROOT、GoPATH、GoModules



下面我们直接写一个最基本 demo


package main

import (
	"github.com/gin-gonic/gin"
	"github.com/thinkerou/favicon"
)

func main() {
	// 创建一个服务
	ginServer := gin.Default()

	ginServer.Use(favicon.New("./favicon.ico"))
	// 访问地址 ，处理请求
	ginServer.GET("/hello", func(context *gin.Context) {
		context.JSON(200, gin.H{"msg": "hello,world"})
	})
	// RESTFUL 风格
	ginServer.GET("/world", World)

	// 服务器端口
	ginServer.Run(":8009")
}

func World(context *gin.Context) {

	context.JSON(200, gin.H{"msg": "hello111,world"})

}



通过浏览器打开 http://127.0.0.1:8009/hello

即可获得数据，这就说明我们的 gin基础访问 已经成功了
```

#### 3:Restful API

```
只要是一个web框架就一定是带有restful 的风格 ，gin也不例外

get post put delete 

在此内部也可以使用post 请求


	ginServer.POST("/user", func(c *gin.Context) {
		c.JSON(200, gin.H{"msg": "hello111,world"})
	})

我们也可以去加载一个静态页面

我们先写一个css 的样式
body{
    background: darkturquoise;
}
在写一个js 的基础模版
alert("加载成功")


我们在先写一个静态页面的html

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>这个是一个测试页面</title>
</head>
<body>
    <h1>这里是一段测试的文字</h1>
    {{.msg}}
</body>
</html>




接下来我们看怎么去加载 只需要一个 ginServer.LoadHTMLGlob 加载静态目录  给出的响应c.HTML即可
然后

package main

import (
	"github.com/gin-gonic/gin"
	"github.com/thinkerou/favicon"
	"net/http"
)

func main() {
	// 创建一个服务
	ginServer := gin.Default()

	ginServer.Use(favicon.New("./favicon.ico"))
	// 访问地址 ，处理请求
	ginServer.GET("/hello", func(context *gin.Context) {
		context.JSON(200, gin.H{"msg": "hello,world"})
	})
	// RESTFUL 风格
	ginServer.GET("/world", World)

	ginServer.POST("/user", func(c *gin.Context) {
		c.JSON(200, gin.H{"msg": "hello111,world"})
	})

	// 加载静态页面
	ginServer.LoadHTMLGlob("templates/*")
	// 加载资源文件
	ginServer.Static("/static", "static")

	// 响应一个页面给前端
	ginServer.GET("/index", func(c *gin.Context) {
		c.HTML(http.StatusOK, "test.html", gin.H{"msg": "这是go传过来的测试数据"})
	})

	// 服务器端口
	ginServer.Run(":8009")
}

func World(context *gin.Context) {

	context.JSON(200, gin.H{"msg": "hello111,world"})

}

所以 静态页面也是可以通过gin 去 执行的 


```

#### 4: 传参

```
gin 的 传参方式也是相对简单的 下面看一个例子
比如 get post restful 或者重定向 表单都可以

	// url?userid=xxx&username=xxx
	ginServer.GET("/user/info", func(context *gin.Context) {
		userid := context.Query("userid")
		context.JSON(http.StatusOK, gin.H{
			"userid":   userid,
			"username": "张三",
		})
	})

	// restful 风格传参
	ginServer.GET("user/info2/:userid/", func(context *gin.Context) {
		userid := context.Param("userid")
		context.JSON(http.StatusOK, gin.H{
			"userid":   userid,
			"username": "张三",
		})
	})

	// 前端给后端传递 json
	ginServer.POST("/json", func(context *gin.Context) {
		// request.body  【】byte 获取到的是切片数据
		data, _ := context.GetRawData()
		var m map[string]interface{}
		// 包装为json 数据 []byte
		_ = json.Unmarshal(data, &m)
		context.JSON(http.StatusOK, m)

	})

	// 支持函数式编程
	ginServer.POST("/user/add", func(context *gin.Context) {

		username := context.PostForm("username")
		password := context.PostForm("password")

		context.JSON(http.StatusOK, gin.H{
			"msg":      "ok",
			"username": username,
			"password": password,
		})

	})
	
	// 404
	
	
	
	// 路由  重定向
	ginServer.GET("/test", func(context *gin.Context) {
		context.Redirect(http.StatusMovedPermanently, "https://wwww.baidu.com")

	})



	// 404 NoRoute
	ginServer.NoRoute(func(context *gin.Context) {
		context.HTML(http.StatusNotFound, "404.html", nil)
	})

	// 路由组
	userGroup := ginServer.Group("/user")
	{
		userGroup.GET("/add")
		userGroup.GET("/login")
		userGroup.GET("/logout")
	}
	orderGroup := ginServer.Group("/order")
	{
		orderGroup.GET("/order")
		orderGroup.GET("/delete")
	}
```

#### 5 自定义中间件 拦截器

```
中间件 也是gin 会自带的一个处理方式 

// 自定义 GO 中间件 拦截器
// 预处理 登陆授权 验证 分页 耗时统计  具体逻辑可以自定义
func myHandler() gin.HandlerFunc {

	return func(context *gin.Context) {
		// 通过自定义 的中间件，设置的值 ，在后续处理只要调用了中间件的都可以拿到这里的参数
		context.Set("usersion", "userid-1")
		context.Next()  // 放行
		context.Abort() // 组织

	}

}


gin 注册使用 一下
	// 注册中间件服务 如果指定就是全局使用
	ginServer.Use(myHandler())
	
// 指定使用 就是直接在 GET 内加参数即可 在路由后面加参数即可 
	ginServer.GET("/test", myHandler(), func(context *gin.Context) {

		usersession := context.MustGet("userSession").(string)
		log.Panicln("=======>>>", usersession)
		context.Redirect(http.StatusMovedPermanently, "https://wwww.baidu.com")

	})

```

#### 6: 多线程

```
func main() {

	// go 携程
	go test()

	for i := 0; i < 1000; i++ {
		fmt.Println("main=>", i)
	}
	time.Sleep(time.Second * 3)
}

func test() {
	for i := 0; i < 1000; i++ {
		fmt.Println("test---->")
	}
}

可以看到 两个循环是有交替执行 的这就是携程 并不会存在一个执行完上一个在执行下一个的等待操作
test---->
test---->
main=> 389
main=> 390
main=> 391

```





