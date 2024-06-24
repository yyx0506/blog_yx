---
title: "Go 函数以及指针"
date: 2023-06-13
categories:
- tranquilpeak
- features
tags:
- golang
- 设计模式
# keywords:
# - 设计模式
# - golang
thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->


### Golang 分析

#### 1：当下Golang的分析

```
优势：
Go 最主要的是极简答的部署方式（可直接编译成机器码，不依赖其他库，直接运行即可部署）
静态型语言（编译的时候可以检查出来隐藏的大多数问题，）
语言层面的并发 天生的基因支持 充分利用多核 
强大的标准库 （runtime 系统调度机制 高效的gc垃圾回收 丰富的标准库）
简单易学 25个关键字 语法简洁 内嵌c语法支持

```

#### 2:const和iota知识点注意事项

```
const 是常量 
const length int = 10
iota 是一个可以配合常量做枚举 的关键字 例如 
const (
	BEIJING = iota // iota = 0
	SHANGHAI  // iota = 1
	SHENZHEN  // iota = 2
)
func main() {
	fmt.Println("jj")
	fmt.Println(SHENZHEN)
}

```

#### 3:Golang 中函数的多返回值三种写法

```
第一种就是匿名的 第二种是给形参名称的 第三种是如果 形参的类型一致可以进行合并的


package main

import "fmt"

func foo1(a string, b int) int {

	fmt.Println("a = ", a)
	fmt.Println("b = ", b)
	c := 100
	return c
}

// 返回值是匿名的
func foo2(a string, b int) (int, int) {
	fmt.Println("a = ", a)
	fmt.Println("b = ", b)

	return 666, 777

}

// 有行参名称的
func foo3(a string, b int) (r1 int, r2 int) {
	fmt.Println("---- foo3 ----")
	fmt.Println(" a = ", a)
	fmt.Println(" b = ", b)
	// r1 r2 属于 foo3的形参 初始化默认的值是 0
	r1 = 1000
	r2 = 2000
	return r1, r2
}

func foo4(a string, b int) (r1, r2 int) {
	fmt.Println("---- foo3 ----")
	fmt.Println(" a = ", a)
	fmt.Println(" b = ", b)
	r1 = 10001
	r2 = 20002
	return r1, r2
}

func main() {

	c := foo1("sss", 877)
	fmt.Println("c = ", c)

	ret1, ret2 := foo2("haha", 999)
	fmt.Println("ret1 = ", ret1, "ret = ", ret2)

}

```

#### 4: import导包路径问题与init方法调用流程

```
Go 语言也是有自己的 init方法和其他的语言一样

一个Go 文件的执行流程是 main --> import --> CONST --> Var --> init() --> main()

类似于下图 

```

![](/img/go12infos1.png)

#### 5:Golang的指针

```
c语言的一大特点就是指针，golang也是有指针的，那么golang 的指针要如何去理解呢，下面来讲述一下指针
下面看一个 程序

package main

import "fmt"

func changeValue(p int) {
	p = 10
}

func main() {
	var a int = 1
	changeValue(a)
	fmt.Println("a = ", a)
}

得到的结果是 a= 1 因为 执行 changeValue时首先开辟了一块内存 现实p=0 然后赋值 p=a 只是将a的值传递了过去并没有传递地址
这个就跟python有些相似 ，但是如果我们想要去改变这个p的值怎么做呢 那就是 改成地址的传递

package main

import "fmt"

func changeValue(p *int) {
	*p = 10
}

func main() {

	var a int = 1
	changeValue(&a)
	fmt.Println("a = ", a)

}
只需要做一下略微的调整即可  加了一个 * 号代表这个类型是一个指针类型 & 表示 吧后面的元素的地址传过去
这样的话a = 10 了
golang 中也是有二级指针的 例如下面的代码

package main

import "fmt"

func main() {
	// 声明一个整型变量a,并赋值为10
	var a int = 10
	// 声明一个整型指针类型变量point，并让他指向a
	var point1 *int = &a
	// 声明一个二级整型指针，即指向一个整型指针变量的指针
	var point2 **int = &point1

	fmt.Println("The address of point2 is ", &point2)
	fmt.Println("The value of point2 is ", point2)
	fmt.Println("The value stored in the memory space pointed to by pointer2 is ", *point2)
	fmt.Println("The address of point1 is ", &point1)
	fmt.Println("The value of point1 is ", point1)
	fmt.Println("The value stored in the memory space pointed to by pointer2 is ", *point1)
	fmt.Println("The value of a is ", a)
	fmt.Println("The address of a is ", &a)
}

```



