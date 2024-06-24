---
title: "Goroutine和channel"
date: 2023-07-07
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

### Goroutine和channel

#### 1:Goroutine基本的模型和调度设计策略

```
这就是go 中的协程 ,一般都是在网络io层去触发

多进程多线程 会发现 开的越多但是效率却并不是成倍的提升，是因为cpu的切换成本太高了，

GO 也对协程进行了处理 将 co routine 改成了 go routine
也就是大家都在说的GMP G-goroutine M-thread P-processor
所以设计一个优秀的调度器才是最应该做的事
调度器的设计策略就是有几大特点 1:复用线程（work stealing 机制 hand off 分离机制） 2:利用并行 3:抢占 4: 全局G队列

```

#### 2:Golang 创建一个 goroutine

```go
// 例如下面这段代码 

package main

import (
	"fmt"
	"time"
)

func task() {
	var t int = 0
	for {
		t++
		fmt.Println(" task 执行的是 t", t)
		time.Sleep(1 * time.Second)
	}

}
// 如果主线程结束的话 那个 从的也会立即结束掉

func main() {
	// 创建一个 goroutine 去执行 task 方法
	go task()
	var i int = 0
	for {
		i++
		fmt.Println(" main 执行的是 i", i)
		time.Sleep(1 * time.Second)
	}
}

```

#### 3: runtime.Goexit()

```go
//当我们执行一个 goroutine 想要其退出的时候可以采用 runtime.Goexit()来退出当前的程序

func main() {
	// 用 go 创建 承载一个形参为空，返回值为空的一个函数
	go func() {
		defer fmt.Println("a.defer")
		func() {
			defer fmt.Println("b.defer")
			//return
			runtime.Goexit()
			fmt.Println("b")
		}()
		//return
		fmt.Println("a")
	}()

	for {
		fmt.Println("执行死循环")
		time.Sleep(1 * time.Second)
	}

}

//这是一个没参数的匿名方法 如果是有参数的我们要怎么做呢

	go func(a, b int) bool {
		fmt.Println("a = ", a, "b = ", b)
		return true
	}(10, 20)

// 例如这样，但是有一个问题 这个return 的方法外层的go如何接收呢 它是一个协程二者之间是并发的形式 所以我们需要借助channel来实现

```

#### 4:channel的基本定义与使用

```
两个goroutine 的通信就是预考 channel 来做的
例如下面的代码。  通过channel 保证了先后顺序 如果主线完成了会等待 此时就会有一个阻塞

func main() {
	// 定义一个 channel
	c := make(chan int)
	go func() {
		defer fmt.Println("goroutine 结束")
		fmt.Println("goroutine 正在运行 ")
		c <- 666 // 将666 发送给c
	}()

	num := <-c // 从c 中接收数据 ，并赋值给num

	fmt.Println("num = ", num)
	fmt.Println("main foroutine 结束")

}

```

#### 5:channel 有缓冲和无缓冲同步问题

```
无缓冲的就是上面的那个例子 只有当两个程序都执行到交互数据时，进行交换，然后两个goroutine 再去做其他的事

而有缓冲的 channel 恰恰相反 发送值的goroutine 可以不等接收值的gotoutine 继续发送 数据会在 channel中进行缓存

我们看一下示例代码

func main() {
	c := make(chan int, 3) // 带有缓存的channel

	fmt.Println("len(c)= ", len(c), ",cap(c)", cap(c))

	go func() {
		defer fmt.Println("子go程结束")

		for i := 0; i < 4; i++ {
			c <- i
			fmt.Println("子 go程正在运行,发送的元素= ", i, ",len()", len(c), ",cap(c) = ", cap(c))

		}

	}()
	time.Sleep(2 * time.Second)

	for i := 0; i < 4; i++ {
		num := <-c // 从c 中接收数据，并赋值给num
		fmt.Println("num = ", num)
	}
	fmt.Println("main 结束")

}
输出是这个 
len(c)=  0 ,cap(c) 3
子 go程正在运行,发送的元素=  0 ,len() 1 ,cap(c) =  3
子 go程正在运行,发送的元素=  1 ,len() 2 ,cap(c) =  3
子 go程正在运行,发送的元素=  2 ,len() 3 ,cap(c) =  3
num =  0
num =  1
num =  2
num =  3
子 go程正在运行,发送的元素=  3 ,len() 3 ,cap(c) =  3
子go程结束
main 结束

可以看出这个就是超出缓存空间后就会等待缓存被取走在下一步 简单来说channel 满了就会阻塞，当channel为空也是会阻塞

```

#### 6:channel的关闭,range, select

```go
func main() {
	c := make(chan int, 5)
	go func() {
		for i := 0; i < 5; i++ {
			c <- i

		}
		close(c) //close 关闭一个channel
	}()
	for {
		// ok 如果为true 表示 channel 没有关闭，如果为false 表示 channel 已经关闭
		// 如果没有 close 的话就会出现死锁的问题
		if data, ok := <-c; ok {
			fmt.Println("data -- ", data)
		} else {
			break
		}
	}
	fmt.Println("main finished ... ")
}

// 如果已经关闭的channel 在发送数据的话 就会报错panic
// channel 一定要通过 make 来进行初始化
	// 也可以使用range 来迭代不断操作channel 效果和上面是一样的
	for data := range c {
		fmt.Println(data)
	}

//当有多个channel 那么如何选择呢？我们使用select关键字来做

```

Go里面提供了一个关键字select，通过select可以监听channel上的数据流动。

有时候我们希望能够借助channel发送或接收数据，并避免因为发送或者接收导致的阻塞，尤其是当channel没有准备好写或者读时。select语句就可以实现这样的功能。

select的用法与switch语言非常类似，由select开始一个新的选择块，每个选择条件由case语句来描述。

与switch语句相比，select有比较多的限制，其中最大的一条限制就是每个case语句里必须是一个IO操作，大致的结构如下：

    select {
    case <- chan1:
        // 如果chan1成功读到数据，则进行该case处理语句
    case chan2 <- 1:
        // 如果成功向chan2写入数据，则进行该case处理语句
    default:
        // 如果上面都没有成功，则进入default处理流程
    }

在一个select语句中，Go语言会按顺序从头至尾评估每一个发送和接收的语句。



如果其中的任意一语句可以继续执行(即没有被阻塞)，那么就从那些可以执行的语句中任意选择一条来使用。



如果没有任意一条语句可以执行(即所有的通道都被阻塞)，那么有两种可能的情况：



l 如果给出了default语句，那么就会执行default语句，同时程序的执行会从select语句后的语句中恢复。



l 如果没有default语句，那么select语句将被阻塞，直到至少有一个通信可以进行下去。



示例代码：

```
package main
 
import (
    "fmt"
)
 
func fibonacci(c, quit chan int) {
    x, y := 1, 1
    for {
        select {
        case c <- x:
            x, y = y, x+y
        case <-quit:
            fmt.Println("quit")
            return
        }
    }
}
 
func main() {
    c := make(chan int)
    quit := make(chan int)
 
    go func() {
        for i := 0; i < 6; i++ {
            fmt.Println(<-c)
        }
        quit <- 0
    }()
 
    fibonacci(c, quit)
}
```

