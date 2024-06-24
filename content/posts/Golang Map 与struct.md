---
title: "Golang Map 与struct"
date: 2023-06-20
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

### Golang Map 与struct

#### 1:map 的声明方式

```
map和slice类似，只不过是数据结构不同，下面是map的一些声明方式。
整体而言 跟slice 很相似 

package main

import (
	"fmt"
)

func main() {
	//第一种声明  相比于slice 就是在声明时加入map 关键字
	var test1 map[string]string
	//在使用map前，需要先make，make的作用就是给map分配数据空间
	test1 = make(map[string]string, 10)
	test1["one"] = "php"
	test1["two"] = "golang"
	test1["three"] = "java"
	fmt.Println(test1) //map[two:golang three:java one:php]
	var test56 map[int]int
	test56 = make(map[int]int, 5)
	test56[3] = 10
	test56[5] = 20
	fmt.Println(test56)
	// map 中括号内是 key 除了 slice,map,function不能作为key外 其他的基本都可以做key

	//第二种声明
	test2 := make(map[string]string)
	test2["one"] = "php"
	test2["two"] = "golang"
	test2["three"] = "java"
	fmt.Println(test2) //map[one:php two:golang three:java]

	//第三种声明
	test3 := map[string]string{
		"one":   "php",
		"two":   "golang",
		"three": "java",
	}
	fmt.Println(test3) //map[one:php two:golang three:java]
	// 声明 map 的 value 也是一个map 类型 随后 给 value对应的map 分配数据空间
	language := make(map[string]map[string]string)
	language["php"] = make(map[string]string, 2)
	language["php"]["id"] = "1"
	language["php"]["desc"] = "php是世界上最美的语言"
	language["golang"] = make(map[string]string, 2)
	language["golang"]["id"] = "2"
	language["golang"]["desc"] = "golang抗并发非常good"

	fmt.Println(language) //map[php:map[id:1 desc:php是世界上最美的语言] golang:map[id:2 desc:golang抗并发非常good]]

	//增删改查
	// val, key := language["php"]  //查找是否有php这个子元素
	// if key {
	//     fmt.Printf("%v", val)
	// } else {
	//     fmt.Printf("no");
	// }

	//language["php"]["id"] = "3" //修改了php子元素的id值
	//language["php"]["nickname"] = "啪啪啪" //增加php元素里的nickname值
	//delete(language, "php")  //删除了php子元素
	fmt.Println(language)
}

一般来说是这三种类型 
但是就会出现一个问题 

	//在 Golang 中，map 是一种关联数据类型，用于存储键值对。在 map 中，键（key）必须是可比较的类型，而且不能是函数、切片、映射等引用类型。 这一块跟 python 完全不一样 
	//这意味着直接使用 int 和 string 作为 map 的键是不可能的，因为它们是不同的类型。
	//然而，你可以通过将 interface{} 类型作为键类型来实现 int 和 string 作为键共存。在 Go 中，interface{} 是所有类型的顶级类型，
	//可以保存任意类型的值。通过使用 interface{} 类型，你可以实现以 int 和 string 为键的 map，但需要在访问 map 中的值时进行类型断言。
package main

import (
	"fmt"
)

func main() {
	// 创建一个 map，键的类型为 interface{}，值为 int
	m := make(map[interface{}]int)

	// 添加 int 类型的键值对
	m[1] = 10

	// 添加 string 类型的键值对
	m["two"] = 20

	// 访问 int 类型的值
	intKey := 1
	intValue, ok := m[intKey]
	if ok {
		fmt.Printf("Value for int key %d: %d\n", intKey, intValue)
	} else {
		fmt.Printf("Value for int key %d not found\n", intKey)
	}

	// 访问 string 类型的值
	stringKey := "two"
	stringValue, ok := m[stringKey]
	if ok {
		fmt.Printf("Value for string key %s: %d\n", stringKey, stringValue)
	} else {
		fmt.Printf("Value for string key %s not found\n", stringKey)
	}

	// 尝试使用错误类型的键
	invalidKey := true
	_, ok = m[invalidKey]
	if !ok {
		fmt.Println("Invalid key type")
	}
}


如果我们遇到key 和 value 都有不同类型的时候那么就可以同时使用 interface 来创建key 和value的类型
package main

import (
	"fmt"
)

func main() {
	// 创建一个 map，键的类型为 interface{}，值的类型为 interface{}
	m := make(map[interface{}]interface{})

	// 添加 int 类型的键值对
	m[1] = 10

	// 添加 string 类型的键值对
	m["two"] = "twenty"

	// 访问 int 类型的值
	intKey := 1
	intValue, ok := m[intKey]
	if ok {
		fmt.Printf("Value for int key %d: %v\n", intKey, intValue)
	} else {
		fmt.Printf("Value for int key %d not found\n", intKey)
	}

	// 访问 string 类型的值
	stringKey := "two"
	stringValue, ok := m[stringKey]
	if ok {
		fmt.Printf("Value for string key %s: %v\n", stringKey, stringValue)
	} else {
		fmt.Printf("Value for string key %s not found\n", stringKey)
	}

	// 尝试使用错误类型的键
	invalidKey := true
	_, ok = m[invalidKey]
	if !ok {
		fmt.Println("Invalid key type")
	}
}

```

#### 2: map 的基本使用方式

```
package main

import "fmt"

func main() {
	cityMap := make(map[string]string)
	// 添加
	cityMap["china"] = "beijing"
	cityMap["japan"] = "tokyo"
	cityMap["usa"] = "Washington"
	// 修改
	cityMap["china"] = "shanghai"

	// 遍历
	for key, value := range cityMap {
		fmt.Println("key:", key, "value:", value)
	}
	// 删除
	delete(cityMap, "china")
	fmt.Println(cityMap)

}

注意的是当map 作为参数传递时 传递是这个map的指针 也就是地址 


```

#### 3:Go struct

```
Golang 是一种面向对象的编程语言，尽管它的面向对象特性相对于一些其他语言可能更为简洁和不那么显眼，但仍然支持面向对象编程。

在 Go 中，面向对象的特性是通过结构体（struct）和方法（method）来实现的。虽然 Go 没有传统的类（class）概念，但你可以通过定义结构体来创建自定义的数据类型，然后在该结构体上定义方法，以实现面向对象的功能。

在Go中 slice 和map 都是引用传递 而 struct 则是值传递 如果想要改变值传递就需要 加入* 变成引用传递 也就是指针


package main

import "fmt"

// 声明一种新的 数据类型 myInt 是 int 的别名

type Myint int

// 定义一个结构体
type Book struct {
	title string
	auth  string
}

func printbook(book Book) {
	fmt.Printf("%v\n", book)

}

func changebook1(book1 Book) {
	book1.title = "C++"
}

func changebook2(book1 *Book) {
	book1.title = "Go"
}

func main() {
	var a Myint = 10
	fmt.Println("a:", a)
	fmt.Printf("type of %T\n", a)

	var book1 Book
	book1.title = "GOlang"
	book1.auth = "zhangsan"
	printbook(book1)
	changebook1(book1)
	printbook(book1)
	changebook2(&book1)
	printbook(book1)

}


```

#### 3: 结构体变成类

```
Go 是没有类的，但是结构体可以完成类的一些操作
例如下面的代码  

package main

import "fmt"

type Hero struct {
	name  string
	ad    int
	level int
}

func (this Hero) GetName() string {
	//fmt.Println(this)
	return this.name
}

func (this Hero) SetName(newName string) {
	this.name = newName
}

func main() {
	var hero1 Hero
	hero1.name = "zhangsan"
	hero1.ad = 10
	hero1.level = 20
	//fmt.Println(hero1)
	name1 := hero1.GetName()
	fmt.Println("name -- ", name1)
	hero1.SetName("lisi")
	name2 := hero1.GetName()
	fmt.Println("name -- ", name2)
}

从代码的静态分析上看是没有问题的，但是执行过后我们就会发现 Setname 并没有改变属性
这是因为 this 是调用该方法的对象的一个副本

所以想要让set 对当前对象生效的话我们需要更改 方法加入* 变成指针即可

package main

import "fmt"

type Hero struct {
	name  string
	ad    int
	level int
}

func (this *Hero) GetName() string {
	//fmt.Println(this)
	return this.name
}

func (this *Hero) SetName(newName string) {
	this.name = newName
}

func main() {
	var hero1 Hero
	hero1.name = "zhangsan"
	hero1.ad = 10
	hero1.level = 20
	//fmt.Println(hero1)
	name1 := hero1.GetName()
	fmt.Println("name -- ", name1)
	hero1.SetName("lisi")
	name2 := hero1.GetName()
	fmt.Println("name -- ", name2)
}


```

