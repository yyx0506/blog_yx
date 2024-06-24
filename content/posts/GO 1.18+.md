---
title: "Go 1.18+"
date: 2023-05-13
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
#### Go 1.18+;

#### 1: 泛型

```
大家都知道 go 之前是不支持泛型的 在1.18后加入了对 泛型的支持

那么泛型是什么？
泛型是一种独立于使用的特定类型编写代码的方式。现在可以编写函数和类型适用于一组类型集合的任何一种。泛型生命周期只在编译期，旨在开发中减少重复代码的编写。

由于go属于静态强类型语言，例如在比较两个数的大小时，没有泛型的时候，仅仅只是传入类型不一样，我们就要再复制一份一样的函数，如果有了泛型就可以减少这类代码。

//int
func GetMaxNumber(a, b int) int {
    if a > b {
        return a
    }
    return b
}
//int32
func GetMaxNumber(a, b int32) int32 {
    if a > b {
        return a
    }
    return b
}

go1.18后引入泛型后,只需要再函数后用中括号声明T可能出现的类型，中间用符号|分隔。

//使用泛型
func GetMaxNumber [T int | int32 ](a, b T) T {
    if a > b {
        return a
    }
    return b
}
```

#### 2:使用泛型

```
1.any: 表示go里面所有的内置基本类型，等价于interface{}
2.comparable: 表示go里面所有内置的可比较类型：int、uint、float、bool、struct、指针等一切可以比较的类型

type Adder interface {
	Add(Adder) Adder
}
type MyInt int
type MyFloat64 float64

func (a MyInt) Add(b Adder) Adder {
	return a + b.(MyInt)
}
func (a MyFloat64) Add(b Adder) Adder {
	return a + b.(MyFloat64)
}
func Sum[T Adder](arr []T) T {
	var sum T = arr[0]
	for _, v := range arr[1:] {
		sum = sum.Add(v).(T)
	}
	return sum
}
func main() {
	intArr := []MyInt{1, 2, 3, 4, 5}
	floatArr := []MyFloat64{1.1, 2.2, 3.3, 4.4, 5.5}
	fmt.Println(Sum(intArr))   // Output: 15
	fmt.Println(Sum(floatArr)) // Output: 16.5
}

在这个例子中，我们定义了Adder接口，该接口要求为它所支持的类型实现一个Add方法。

然后我们创建了两个自定义类型，MyInt和MyFloat64，它们实现了Adder接口。

为这两个类型定义了Add方法，Sum函数使用Add方法进行加法运算。
```

#### 3:unsafe 包中新增函数原型

```
package main

import (
    "fmt"
    "unsafe"
)

func main() {
    arr := [5]int{1, 2, 3, 4, 5}
    // 注意: 这里是传递首个元素的指针，而非数组的指针
    sli := unsafe.Slice(&arr[0], 3)
    fmt.Printf("slice = %+v, len = %d\n", sli, len(sli))

    // 修改切片元素的值会影响底层的数组
    for i := range sli {
        sli[i] *= 100
    }
    fmt.Printf("slice = %+v, len = %d\n", sli, len(sli))
    // 此时数组的元素值已经变化
    fmt.Printf("array = %+v, len = %d\n", arr, len(arr))
}


// 运行输出如下
// slice = [1 2 3], len = 3
// slice = [100 200 300], len = 3
// array = [100 200 300 4 5], len = 5



package main

import (
    "fmt"
    "unsafe"
)

func main() {
    bs := []byte{'h', 'e', 'l', 'l', 'o'}
    // 注意: 这里是传递首个元素的指针，而非切片的指针
    str := unsafe.String(&bs[0], 2)
    fmt.Printf("str = %s, len = %d\n", str, len(str))

    // 修改参数指向的值会同时修改字符串的值
    bs[1] = 'i'
    
    // 此时字符串的值已经变化
    fmt.Printf("str = %s, len = %d\n", str, len(str))
    fmt.Printf("bs = %s, len = %d\n", bs, len(str))
}

// 运行输出如下
// str = he, len = 2 
// str = hi, len = 2 
// bs = hillo, len = 2
```

#### 4 标准库的增强

```
• 新增了几个 时间转换格式常量
• 新包 crypto/ecdh 支持通过 NIST 曲线和 Curve25519 椭圆曲线 Diffie-Hellman 密钥交换
• 新类型 http.ResponseController 访问 http.ResponseWriter 接口未处理的扩展请求
• httputil.ReverseProxy 包含一个新的 Rewrite 钩子函数，取代了之前的 Director 钩子
• 新方法 context.WithCancelCause 提供了一种方法来取消具有给定错误的上下文
• os/exec.Cmd 结构体中的新字段 Cancel 和 WaitDelay, 指定 Cmd 在其关联的 Context 被取消或其进程退出时的回调

新方法 errors.Join 返回一个错误列表，如果错误类型实现了 Unwrap() []error，则可以再次获得错误列表
在原有 Go1.13 的 errors API 上进行新增和修改，核心是支持一个错误可以封装多个错误的特性。新方法 errors.Join 返回一个错误列表，如果错误类型实现了 Unwrap() []error，则可以再次获得错误列表。

func main() {
 err1 := errors.New("err1")
 err2 := errors.New("err2")
 err := errors.Join(err1, err2)
 fmt.Println(err)
 if errors.Is(err, err1) {
  fmt.Println("err is err1")
 }
 if errors.Is(err, err2) {
  fmt.Println("err is err2")
 }
}
// 输出结果
err1
err2
err is err1
err is err2


优化时间比较和格式记忆
Go1.20 新增了几个 时间转换格式常量，便于直接引用：
package time

const (
    DateTime   = "2006-01-02 15:04:05"
    DateOnly   = "2006-01-02"
    TimeOnly   = "15:04:05"
)
```

