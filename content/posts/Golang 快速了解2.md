---
title: "Go defer 数组以及切片"
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


### Golang 快速了解2

#### 1:defer语句的调用顺序

```
这个关键字在其他语言中很少出现，我们看一下这个关键字的作用是什么

他有点类似于 python 的 final 字段
例如
package main

import "fmt"

func main() {
	// 声明一个整型变量a,并赋值为10
	var a int = 10
	// 声明一个整型指针类型变量point，并让他指向a
	var point1 *int = &a
	// 声明一个二级整型指针，即指向一个整型指针变量的指针
	var point2 **int = &point1
	defer fmt.Println("end")
	fmt.Println("The address of point2 is ", &point2)
	fmt.Println("The value of point2 is ", point2)
	fmt.Println("The value stored in the memory space pointed to by pointer2 is ", *point2)
	fmt.Println("The address of point1 is ", &point1)
	fmt.Println("The value of point1 is ", point1)
	fmt.Println("The value stored in the memory space pointed to by pointer2 is ", *point1)
	fmt.Println("The value of a is ", a)
	fmt.Println("The address of a is ", &a)
}

他会在这个函数执行完后在执行
当出现多个defer 时 就会最后面的先执行 这就是压栈

当出现 return 时 时谁先谁后呢 看下面的代码


func deferFunc() int {
	fmt.Println(" defer func called")
	return 0
}

func retutnFunc() int {
	fmt.Println(" return func called")
	return 0
}

func returnAnddefer() int {
	defer deferFunc()
	return retutnFunc()
}

func main() {
	returnAnddefer()
}
执行后发现的就是。return 执行 defer 最后执行 就是当这个函数的生命周期结束后才会调用


```

#### 2:Golang 数组和动态数组的区别

```
我们先声明两个数组 并便利一下

func main() {
	var myArray1 [10]int
	for i := 0; i < 10; i++ {
		fmt.Println(myArray1[i])
	}

	myArray2 := [10]int{1, 2, 3, 4, 5}
	for index, value := range myArray2 {
		fmt.Println("index = .", index, " value = ", value)
	}
}

如果我们声明 一个动态数组呢 就是 slice 传说的 切片

func printArray(myArray []int) {
	// 引用传递
	// _ 表示匿名的变量
	for _, value := range myArray {
		fmt.Println("value = ", value)
	}
	myArray[1] = 100

}

func main() {
	myArray := []int{1, 2, 3, 4}
	fmt.Printf("myarray types is %T\n", myArray)
	printArray(myArray)
	fmt.Println("======")

	for _, value := range myArray {
		fmt.Println("value = ", value)
	}

}

普通的数组是值传递 而动态数组 是引用传递 这个有点类似于python了


```

#### 3:slice 的声明定义方式 以及切片追加与截取

```
一般来说是有四种方式去声明 判断一个slice 是否是一个空切片 只需要 判读其是否等于nil 即可
func main() {
	// 声明slice1 是一个切片 并且初始化 默认数据是1，2，3 长度是3
	var slice1 = []int{1, 2, 3}
	fmt.Println("slice1 = ", slice1)
	// 声明 slice2 是一个切片 但是并没有 给slice 分配空间 通过make 来创建一个空间
	var slice2 []int
	slice2 = make([]int, 3) // 开辟空间 以及 长度
	fmt.Println("slice2 = ", slice2)

	// 第三种方式 合二为一
	var slice3 = make([]int, 3)
	fmt.Println("slice3 = ", slice3)
	
	// 第四种直接 ：= 
	slice4 := make([]int, 3)
	fmt.Println("slice4 = ", slice4)
	
	if slice1 == nil {
		fmt.Println("slice1 是一个空切片")
	}else {
		fmt.Println("slice1 不是一个空切片")
	}
}

切片的 使用方式。切片的追加与 截取

func main() {
	// 第二个参数是长度，第二个参数是容量
	var numbers = make([]int, 3, 5)

	fmt.Printf(" len = %d ,cap = %d ,slice = %v\n", len(numbers), cap(numbers), numbers)
	// 向 numbers 切片 追加一个元素1 ，numbers len = 4 , [0,0,0,1] ,cap = 5

	numbers = append(numbers, 1)
	fmt.Printf(" len = %d ,cap = %d ,slice = %v\n", len(numbers), cap(numbers), numbers)

	numbers = append(numbers, 2)
	fmt.Printf(" len = %d ,cap = %d ,slice = %v\n", len(numbers), cap(numbers), numbers)

	//  向一个cap 容量已满的 sliced 追加元素 会新开辟一个指定cap 大小的容量
	numbers = append(numbers, 3)
	fmt.Printf(" len = %d ,cap = %d ,slice = %v\n", len(numbers), cap(numbers), numbers)
	
	// 如果不指定容量 的话会根据 生成的长度为容量去开辟一个新的内存 
}

通过 这段代码我们可以得到 切片的长度和容量不同，长度表示左指针和右指针之间的距离，容量表示左指针至底层数组末尾的距离

切片的扩容机制，append 的时候 如果长度增加超过容量，那么容量增加两倍

func main() {
	s := []int{1, 2, 3} //len = 3, cap = 3, [1,2,3]

	//[0, 2)
	s1 := s[0:2] // [1, 2]

	fmt.Println(s1)

	s1[0] = 100

	fmt.Println(s)
	fmt.Println(s1)

	//copy 可以将底层数组的slice一起进行拷贝
	s2 := make([]int, 3) //s2 = [0,0,0]

	//将s中的值 依次拷贝到s2中
	copy(s2, s)
	fmt.Println(s2)

}

这个有点像python的写法

左开右闭  如果改变了其中的值会发现 原有的结果也会发生变化 说明 切片即是截取的也是地址而已并没有新开辟一块地址(这一点和python不一样)
如果想要有新的空间去开辟的话 可以使用copy 这一点跟python 类似
这个切片有点类似于 python 的写法一样


```

