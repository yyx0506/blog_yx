---
title: "Golang  流程控制"
date: 2022-08-22
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


### Golang 3

#### 1: Go 的算数运算符

- +：相加
- -：相减
- *：相乘
- /：相除
- %：求余

在golang中， ++ 和 -- 只能单独使用，错误的写法如下

```
var i int = 8
var a int
a = i++  // 错误，i++只能单独使用
a = i--  // 错误，i--只能单独使用
```

同时在golang中，没有 ++i这样的操作

```
var i int = 1
++i  // 错误
正确的写法
var i int = 1
i++ //正确
```

#### 2: Go 的流程控制

流程控制是每种编程语言控制逻辑走向和执行次序的重要部分，流程控制可以说是一门语言的“经脉"

Go 语言中最常用的流程控制有if和for，而switch和goto主要是为了简化代码、降低重复代码而生的结构，属于扩展类的流程控制。

```
if else  


func main() {
	var num = 10
	if num == 10 {
		fmt.Println("hello == 10")
	} else if(num > 10) {
		fmt.Println("hello > 10")
	} else {
		fmt.Println("hello < 10")
	}
}


还有另外一种写法
if num2:= 10; num2>=10 {
    fmt.Println("hello >=10")
}

for 循环结构  
Go语言中的所有循环类型均可使用for关键字来完成
for循环的基本格式如下：

for 初始语句; 条件表达式; 结束语句 {
	循环体
}
这个写法非常类似 java 的写法

条件表达式返回true时循环体不停地进行循环，直到条件表达式返回false时自动退出循环
实例：打印1 ~ 10

for i := 0; i < 10; i++ {
    fmt.Printf("%v ", i+1)
}
注意，在Go语言中，没有while语句，我们可以通过for来代替
for {
    循环体
}
for循环可以通过break、goto、return、panic语句退出循环
```

#### 3: for range（键值循环）

Go 语言中可以使用for range遍历数组、切片、字符串、map及通道（channel）。通过for range遍历的返回值有以下规律：

- 数组、切片、字符串返回索引和值。
- map返回键和值。
- 通道（channel）只返回通道内的值。

实例：遍历字符串

```
var str = "你好golang"
for key, value := range str {
    fmt.Printf("%v - %c ", key, value)
}
结果如下（go 中 中文占 3个字节 字母占一个字节 ）
0 - 你 3 - 好 6 - g 7 - o 8 - l 9 - a 10 - n 11 - g 
```

遍历切片（数组）

```
var array = []string{"php", "java", "node", "golang"}
for index, value := range array {
    fmt.Printf("%v %s ", index, value)
}
结果如下 
0 php 1 java 2 node 3 golang 

```

#### 4: switch case

使用switch语句可方便的对大量的值进行条件判断 例如

```
extname := ".a"
switch extname {
	case ".html": {
		fmt.Println(".html")
		break
	}
	case ".doc": {
		fmt.Println(".doc")
		break
	}
	case ".js": {
		fmt.Println(".js")
	}
	default: {
		fmt.Println("其它后缀")
	}
}
```

switch的另外一种写法

```
switch extname := ".a"; extname {
	case ".html": {
		fmt.Println(".html")
		break
	}
	case ".doc": {
		fmt.Println(".doc")
		break
	}
	case ".js": {
		fmt.Println(".js")
	}
	default: {
		fmt.Println("其它后缀")
	}
}
```

同时一个分支可以有多个值

```
extname := ".txt"
switch extname {
	case ".html": {
		fmt.Println(".html")
		break
	}
	case ".txt",".doc": {
		fmt.Println("传递来的是文档")
		break
	}
	case ".js": {
		fmt.Println(".js")
	}
	default: {
		fmt.Println("其它后缀")
	}
}
```

tip：在golang中，break可以不写，也能够跳出case，而不会执行其它的。

如果我们需要使用switch的穿透 fallthrought，fallthrough语法可以执行满足条件的 case 的下一个case，为了兼容c语言中的case设计 

```
extname := ".txt"
switch extname {
extname := ".txt"
	switch extname {
	case ".html":
		fmt.Println(".html")
	case ".txt", ".doc":
		fmt.Println("传递来的是文档")
		fallthrough
	case ".js":
		fmt.Println(".js")
	default:
		fmt.Println("其它后缀")
	}
}
```

fallthrough 只能穿透紧挨着的一层，不会一直穿透，但是如果每一层都写的话，就会导致每一层都进行穿透

#### 5: break：跳出循环

Go语言中break 语句用于以下几个方面：

- 用于循环语句中跳出循环，并开始执行循环之后的语句。
- break在switch（开关语句）中在执行一条case后跳出语句的作用。
- 在多重循环中，可以用标号label标出想break的循环。

```
var i = 0
for  {
    if i == 10{
        fmt.Println("跳出循环")
        break
    }
    i++
    fmt.Println(i)
}

go：跳转到指定标签

goto 语句通过标签进行代码间的无条件跳转。goto 语句可以在快速跳出循环、避免重复退出上有一定的帮助。Go语言中使用goto语句能简化一些代码的实现过程。

var n = 20
	if n > 24 {
		fmt.Println("成年人")
	} else {
		goto lable3
	}

	fmt.Println("aaa")
	fmt.Println("bbb")
lable3:
	fmt.Println("ccc")
	fmt.Println("ddd")
```

