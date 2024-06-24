---
title: "Golang hu 基础"
date: 2022-07-31
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


### Golang 1

#### 1:Go 介绍

```
2007年，谷歌工程师Rob Pike，Ken Thompson和Robert Griesemer开始设计一门全新的语言，这是Go语言的最初原型。
2009年11月10日，Google将Go语言以开放源代码的方式向全球发布。
2015年8月19日，Go1.5版发布，本次更新中移除了”最后残余的c代码”
2018年2月16日，Go语言Go1.10版发布。
Go语言保证了既能到达静态编译语言的安全和性能，又达到了动态语言开发维护的高效率，使用一个表达式来形容Go语言：Go=C+Python，说明Go语言既有C静态语言程序的运行速度，又能达到Python动态语言的快速开发。
```

#### 2:Go 的安装

```
之前由于工作原因有一个服务用python写的效率很低下，并发不太行，所以选择了go作为开发，作为一个完全不懂go的人第一步就是要 做到装ida
推荐 https://www.macwk.com/article/jetbrains-crack  只针对mac 
第二步是装环境 go的网站老是打不开 推荐 国内的一个镜像网站安装 https://studygolang.com/dl  选择适合自己电脑的版本
一路傻瓜式安装， 然后创建一个项目。
```

#### 3：第一个go程序

```
创建完项目后我们直接开始书写 第一个项目 因为有其他语言的基础 所以我们也是快速的上手
package main
import "fmt"

func main() {
	fmt.Println("hello world!")
}

代码的说明

- go文件的后缀是.go
- package main：表示该hello.go文件所在的包是main，在go中，每个文件都归属与一个包
- import "fmt"：表示引入一个包，可以调用里面的函数
- func main()：表示程序入口，是一个主函数
```

#### 4 ： go 变量与常量

```
跟很多语言一样 go 也是有自己变量与常量 而且基本都大同小异 
Go语言变量是由字母、数字、下划线组成，其中首个字符不能为数字。Go语言中关键字和保留字都不能用作变量名
Go语言中变量需要声明后才能使用，同一作用域内不支持重复声明。并且Go语言的变量声明后必须使用。
变量声明后，没有初始化，打印出来的是空
定义变量 (两种方式):
var name = "zhangsan"
var name string = "zhangsan"
或者采用短变量模式  变量名 := 表达式（短变量只能用于 局部变量，不能用于全局变量声明 ）
声明多个变量 
func main() {
	var a1, a2 string
	a1 = "123"
	a2 = "123"
	fmt.Println(a1)
	fmt.Println(a2)
}

关于常量。常量是恒定不变的值。定义时需要使用到 const 关键字
const pi = 3.14
// 定义两个常量
const(
    A = "A"
    B = "B"
)
// const同时声明多个常量时，如果省略了值表示和上面一行的值相同
const(
    A = "A"
    B
    C
)
```

#### 5 Const常量结合iota的使用

```
iota是golang 语言的常量计数器，只能在常量的表达式中使用

iota在const关键字出现时将被重置为0（const内部的第一行之前），const中每新增一行常量声明将使iota计数一次（iota可理解为const语句块中的行索引）。

每次const出现，都会让iota初始化为0【自增长】
const a = iota // a = 0
const (
	b = iota // b=0
    c        // c = 1
    d        // d = 2
)
const  iota使用_跳过某些值
const (
	b = iota // b=0
    _
    d        // d = 2
)
```

