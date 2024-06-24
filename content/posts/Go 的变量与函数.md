
---
title: "Go 的变量与函数"
date: 2023-05-02
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

#### Go 的变量与函数

#### 1:变量的定义

```
任何的语言都有变量 go也不例外

go 是静态型语言 

var (
	name1 string = "sss"
	age          = 18
	name2 string
	age2  int
)

func main() {
	fmt.Println("hello world!")
	fmt.Println(name1)
	fmt.Println(age, reflect.TypeOf(age))
	fmt.Println(name2, reflect.TypeOf(name2))
	fmt.Println(age2, reflect.TypeOf(age2))
}

------输出------
hello world!
sss
18 int
 string
0 int


可以看到 var 我们声明一个全局变量的时候 可以给类型 或者不给 go语言会通过 变量值 检测出当前变量的类型 是一个int
如果需要自己指定变量类型的值类型，可以自己用函数转化
var age int = 18

当我们没有给默认值时 string 类型 默认为空 int 类型默认为 0 当不给默认值时 必须带上类型 否则就会报错

当我们定义常量时使用 const 关键字 语法与var 一样

除了以上的声明方式外 还有

ccc := 333
fmt.Println(ccc)
省略var,注意:=左侧的变量不应该是已经声明过的，否则会导致编译错误【变量的生命周期为函数体类】

```

#### 2:变量的交换

```
现在我们有 a,b 两个值 a = 100 b = 200
在很多语言中 这二者是无法进行交换的
go 语言是如何去做的呢

func main() {
	var a, b int = 100, 200
	a, b = b, a
	fmt.Println(a, b)
}
这样的做法很像 python 的做法

a , b = 1,34
a,b = b,a
print(a,b)

只不过 go 的声明需要比python 更加的复杂一些


```

#### 3:匿名变量

```
这个是go语言的一个特性 python 中有类似的是匿名函数

func test() (int, int) {
	var a, b int = 100, 200
	return a, b
}

func main() {
	var a, b = test()
	fmt.Println(a, b)
	
	var a2,_ = test()
	_,b2 := test()
	fmt.Println(a2,b2)
	
}

例如我们声明了一个方法返回两个 int 值

去接收这两个int 值时如果有我们不需要的返回值可以使用 _ 来代理 这样他们就不会占用内存空间 直接被丢弃

```

#### 4:变量的作用域

```
go 语言编译检查会非常严格 如果声明了变量而不使用会直接报错 所以一定要理解变量的作用域 

变量只有 局部变量和 全部变量

局部变量 在函数体内的时局部变量 它遵循着就近原则的。 

函数外部定义的是 全局变量。

```

#### 5:函数的本质

```
函数的本身就是一个数据类型。
func 不加括号 函数就是一个变量 如果加了括号 就成了函数的调用。他是引用类型的
函数名（）：调用的是返回的结果 函数名：指向函数体的内存地址，一种特殊类型的指针变量

匿名函数 在很多语言都有 匿名函数 go也是一样的
例如 匿名函数 可以有参数也可以有返回值 就像正常的函数一样 不过 他不需要像python一样用到 lambda 关键字

func main() {
	c := func(a, b int) int {
		fmt.Println("sss", a, b)
		return a + b
	}(4, 5)
	fmt.Println(c)
}

总结就是 go语言支持函数式编程： 1 将匿名函数作为另外一个函数的参数，回调函数 2 将匿名函数作为另外一个函数的返回值 可以形成闭包结构

```

高阶函数：根据go语言的数据类型特点 ，可以将一个函数作为另外一个函数的参数。

func1()  func2()

将 func1() 函数作为 func2 这个函数的参数

func2函数：叫做高阶函数，接收了一个函数作为参数的函数

func1函数：叫做回调函数，作为另外一个函数的参数

例如下面的

```
func add(a, b int) int {
	return a + b
}

func main() {
	result := add(1, 4)
	fmt.Println(result)

	result2 := oper(4, 5, add)
	fmt.Println(result2)
}

func oper(a, b int, f func(int, int) int) int {
	result := f(a, b)
	return result
}

```

下面说一下闭包 这个在很多语言中也是有的

```
简单来说就是 一个外层函数中，有内层函数，该内层函数中，会操作外层函数的局部变量 
并且该外层函数的返回值就是这个内层函数。
这个内层函数和外层函数的局部变量，统称为闭包结构

局部变量的生命周期就会发生改变，正常的局部变量回随着函数的调用而创建，随着函数的结束而销毁
但是闭包结构中的外层函数的局部变量并不会随着外层函数的结束而销毁，因为内层函数还在继续使用

例如

func incr() func() int {
	// 局部变量 i
	i := 0
	// 定义一个匿名函数 给变量自增并返回
	fun := func() int { // 内层函数，没有执行的
		// 局部变量的生命周期发生了变化
		i++
		return i
	}
	return fun
}

func main() {
	r1 := incr()
	fmt.Println(r1)
	v1 := r1()
	fmt.Println(v1)
	v2 := r1()
	fmt.Println(v2)
	v3 := r1()
	fmt.Println(v3)
	v4 := r1()
	fmt.Println(v4)

	r2 := incr()
	v55 := r2()
	fmt.Println(v55)
	fmt.Println(r1())
}


0x108d4e0
1
2
3
4
1
5


结果就充分说明了闭包的含义
```

