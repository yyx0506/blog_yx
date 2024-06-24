---
title: "Go 多态 接口"
date: 2023-07-02
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


### Go 多态 接口

#### 1:多态是什么？

```
面向对象语言都是有多态的概念 例如python
我们看一段简单的python代码  其实就是 相同的方法名 但是却有不同的实现 最后都是走了自己的方法

class Animal:
    def speak(self):
        return "Unknown animal sound"

class Dog(Animal):
    def speak(self):
        return "Woof!"

class Cat(Animal):
    def speak(self):
        return "Meow!"

def animal_sound(animal):
    return animal.speak()

# 创建不同的动物对象
animal1 = Animal()
animal2 = Dog()
animal3 = Cat()

# 多态性的体现：相同的方法名，不同的实现
print(animal_sound(animal1)) # 输出: Unknown animal sound
print(animal_sound(animal2)) # 输出: Woof!
print(animal_sound(animal3)) # 输出: Meow!


那么GO 语言是如何去实现这个功能的呢 这就需要引入一个新的点 interface
package main

import "fmt"

// 本质是一个指针
type AnimalTF interface {
	Sleep()
	GetColor() string
	GetType() string
}

// 具体的 struct
type Cat struct {
	color string //
}

func (this *Cat) Sleep() {
	fmt.Println("cat  is  Sleep")
}

func (this *Cat) GetColor() string {
	return this.color
}

func (this *Cat) GetType() string {
	return "Cat"
}

type Dog struct {
	color string //
}

func (this *Dog) Sleep() {
	fmt.Println("dog  is  Sleep")
}

func (this *Dog) GetColor() string {
	return this.color
}

func (this *Dog) GetType() string {
	return "Dog"
}

func showanimal(animal AnimalTF) {
	animal.Sleep()
	fmt.Println("color is ", animal.GetColor())
	fmt.Println("type is ", animal.GetType())
}

func main() {
	// 第一种方式
	var animal AnimalTF // 定义一个接口的数据类型 父类的指针
	animal = &Cat{"Green"}
	animal.Sleep()

	var animal1 AnimalTF
	animal1 = &Dog{"Yellow"}
	animal1.Sleep()

	//第二种方式
	cat := Cat{"red"}
	dog := Dog{"black"}
	showanimal(&cat)
	showanimal(&dog)
}

可以看出来 GO语言中的多态性主要依赖于接口（Interfaces）和方法（Methods）的结合。通过定义接口，并让不同的类型实现这些接口，可以在调用相同的方法时实现不同的行为。这样，就可以达到类似多态的效果。

```

#### 2: struct 两种实现方法的区别

```
来看一段代码

type Dog struct{}

func (d Dog) Speak() string {
	return "Woof!"
}


type Dog struct{}

func (this *Dog) Speak() string {
	return "Woof!"
}


两种写法中，都定义了 Dog 结构体并实现了 Speak() 方法，但它们之间有一个细微的差别：方法的接收者。

第一种写法：
在这种写法中，Speak() 方法的接收者是 Dog 结构体的一个值类型（非指针类型）。这意味着在调用 Speak() 方法时，Go 语言会对 Dog 类型的对象进行值拷贝，方法中对对象的修改不会影响原始对象。这种方式适用于不需要修改对象状态的情况，例如只是返回一些信息。

第二种写法：
在这种写法中，Speak() 方法的接收者是 Dog 结构体的指针类型。这意味着在调用 Speak() 方法时，Go 语言不会对 Dog 类型的对象进行拷贝，而是直接传递对象的指针。因此，方法中对对象的修改会影响原始对象。这种方式适用于需要修改对象状态的情况。

两种写法中的 Speak() 方法的功能都是一样的，只是接收者类型不同，因此在使用时需要根据具体情况来选择合适的写法。

如果你的 Speak() 方法不需要修改 Dog 对象的状态，那么使用第一种写法是更合适的选择。如果 Speak() 方法需要修改 Dog 对象的状态，那么使用第二种写法是更合适的选择。

似乎看不出来区别 那我们用一个具体一点的例子来看

package main

import "fmt"

type Dog struct {
    Name string
}

// 非指针接收者方法
func (d Dog) ChangeNameNonPointer(newName string) {
    d.Name = newName
}

// 指针接收者方法
func (d *Dog) ChangeNameWithPointer(newName string) {
    d.Name = newName
}

func main() {
    // 创建一个 Dog 对象
    dog1 := Dog{Name: "Buddy"}

    // 使用非指针接收者方法，不会修改原始对象
    dog1.ChangeNameNonPointer("Max")
    fmt.Println("dog1 (Non-pointer receiver):", dog1.Name) // 输出: Buddy

    // 使用指针接收者方法，会修改原始对象
    (&dog1).ChangeNameWithPointer("Max")
    fmt.Println("dog1 (Pointer receiver):", dog1.Name) // 输出: Max

    // 创建一个指向 Dog 对象的指针
    dog2 := &Dog{Name: "Lucy"}

    // 使用非指针接收者方法，不会修改原始对象
    dog2.ChangeNameNonPointer("Molly")
    fmt.Println("dog2 (Non-pointer receiver):", dog2.Name) // 输出: Lucy

    // 使用指针接收者方法，会修改原始对象
    dog2.ChangeNameWithPointer("Molly")
    fmt.Println("dog2 (Pointer receiver):", dog2.Name) // 输出: Molly
}

在上面的例子中，我们定义了一个 Dog 结构体，其中包含一个 Name 字段。然后，我们分别定义了 ChangeNameNonPointer 方法和 ChangeNameWithPointer 方法，前者使用非指针接收者，后者使用指针接收者。

在 main 函数中，我们创建了两个 Dog 对象 dog1 和 dog2，一个是值类型对象，一个是指针类型对象。

首先，我们使用非指针接收者方法 ChangeNameNonPointer 尝试修改 dog1 和 dog2 的名字。但是，由于使用了非指针接收者，这两个对象的名字没有被修改，输出的名字依然是原始的值。

接着，我们使用指针接收者方法 ChangeNameWithPointer 尝试修改 dog1 和 dog2 的名字。由于使用了指针接收者，这两个对象的名字被成功修改了。

这个例子清楚地展示了使用不同接收者类型时的区别。非指针接收者方法不会对原始对象进行修改，而指针接收者方法可以修改原始对象。因此，在选择接收者类型时，需要根据方法是否需要修改对象的状态来进行判断。如果需要修改对象状态，使用指针接收者；如果不需要修改对象状态，使用非指针接收者即可。

```

#### 3:interface (万能类型)

```
interface 空接口 以及类型断言机制

package main

import "fmt"

// interface() 是万能数据类型
func myFunc(arg interface{}) {

	fmt.Println("myFunc is called")
	fmt.Println(arg)
	// interface{} 该如何区分 此时引用的底层数据类型是什么？
	//给 interface{} 提供 “断言” 的机制
	value, ok := arg.(string)
	if !ok {
		fmt.Println("arg is not string type")
	} else {
		fmt.Println("arg is string type")
		fmt.Printf("value type is %T\n", value)
	}
}

type Book struct {
	auth string
}

func main() {
	book := Book{"Golang"}
	myFunc(book)
	myFunc(1000)
	myFunc("111")

}
```

#### 4:Pair

```
在Golang 中一个变量是含有 type 和 value 两个属性的。其中type 又分为static type 和 concrete type
这两个合在一起 称之为 pair对

```



