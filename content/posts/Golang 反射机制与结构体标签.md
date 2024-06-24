---
title: "Golang 反射机制与结构体标签"
date: 2023-07-03
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


### Golang 反射机制与结构体标签

#### 1:reflect


Go 语言中的 `reflect` 包是一个强大的内置包，用于在运行时操作对象的类型信息，反射对象和值。它允许程序在编译时不知道具体类型的情况下，动态地检查和操作对象的属性、方法和类型。

`reflect` 包的主要类型和函数如下：

1. `reflect.Type`：表示 Go 语言中的类型，可以获取类型的名称、种类、字段、方法等信息。
   - `TypeOf(interface{}) reflect.Type`：返回给定对象的类型。
   - `Kind()`：获取类型的种类，例如是 `int`、`string`、`struct` 等。
   - `NumField()`：获取结构体类型的字段数量。
   - `Field(int) reflect.StructField`：获取结构体类型的指定字段信息。
   - `NumMethod()`：获取类型的方法数量。
   - `Method(int) reflect.Method`：获取类型的指定方法信息。
2. `reflect.Value`：表示 Go 语言中的值，可以获取和设置值的内容。
   - `ValueOf(interface{}) reflect.Value`：返回给定对象的值。
   - `Interface()`：将 `Value` 转换为 `interface{}` 类型。
   - `IsValid()`：检查 `Value` 是否有效，用于检查是否返回了空值。
   - `Kind()`：获取值的种类，例如是 `int`、`string`、`struct` 等。
   - `Int()`、`Float()`、`String()`、`Bool()`：将值转换为对应的基本类型。
   - `Field(int) reflect.Value`：获取结构体类型的指定字段的值。
   - `Method(int) reflect.Value`：调用结构体类型的指定方法。
   - `Set()`：设置值的内容。

通过 `reflect` 包，可以在运行时获取对象的类型信息并进行相关操作，这对于实现通用函数、反射配置文件或实现一些动态的编程任务非常有用。但需要注意，反射的使用通常会降低程序的性能，因为它需要在运行时进行类型检查和转换。因此，在性能要求较高的场景中应该避免过度使用反射。

以下是一个简单的示例，展示如何使用 `reflect` 包来获取类型信息和动态调用方法：

```go
package main

import (
	"fmt"
	"reflect"
)

type Person struct {
	Name string
	Age  int
}

func (p Person) SayHello() {
	fmt.Printf("Hello, my name is %s and I am %d years old.\n", p.Name, p.Age)
}

func main() {
	p := Person{"Alice", 30}

	// 使用 reflect 获取对象的类型和值
	pType := reflect.TypeOf(p)
	pValue := reflect.ValueOf(p)

	// 获取对象的类型名称和字段数量
	fmt.Println("Type:", pType.Name())         // 输出: Type: Person
	fmt.Println("NumField:", pType.NumField()) // 输出: NumField: 2

	// 获取对象的字段名和值
	for i := 0; i < pType.NumField(); i++ {
		field := pType.Field(i)
		value := pValue.Field(i).Interface()

		fmt.Printf("Field Name: %s, Field Value: %v\n", field.Name, value)
	}

	// 使用 reflect 调用对象的方法
	pMethod := pType.Method(0) // 获取第一个方法
	pValue.Method(0).Call(nil) // 调用第一个方法，无参数
	fmt.Printf("pMethod -- ", pMethod)

	// 动态创建实例
	newP := reflect.New(pType).Elem()
	newP.Field(0).SetString("Bob")
	newP.Field(1).SetInt(25)

	// 将新创建的实例转换为原始类型，并调用方法
	newPerson := newP.Interface().(Person)
	newPerson.SayHello() // 输出: Hello, my name is Bob and I am 25 years old.
}
在上述示例中，我们创建了一个名为 Person 的结构体，并定义了一个方法 SayHello。然后，使用 reflect 包来获取结构体的类型信息和值。我们展示了如何获取类型名称、字段信息以及字段的值，并通过 reflect 包动态地调用了结构体的方法。

最后，我们还展示了如何使用 reflect 包来动态创建一个新的结构体实例，并将其转换为原始类型，并且调用了新实例的方法。

需要注意的是，reflect 包中的一些方法需要小心使用，特别是涉及到类型转换、赋值和调用方法时。正确的使用反射可以带来很多灵活性和便利性，但也需要额外的小心和测试，以确保代码的正确性和性能。


所以关于使用 reflect 的建议是

在以下情况下，可能会使用 reflect 包：

通用性函数：在某些情况下，无法确定要处理的具体类型，或者希望编写通用的处理函数，可以使用反射来实现这样的功能。例如，序列化和反序列化库常常使用反射来处理未知类型的数据。

配置解析：在需要动态解析配置文件或将配置信息映射到不同的数据结构时，反射可以帮助实现这样的功能。

RPC 和框架：某些 RPC 和框架需要在运行时动态地解析传递的数据，这时反射可能会派上用场。

尽管在上述情况下反射可能会有用，但是在一般的应用程序中，大多数情况下都不需要使用反射。使用反射会增加代码的复杂性，并且会影响程序的性能。在许多情况下，使用静态类型更为简单和高效，所以在编写 Golang 程序时应该尽量避免过度使用反射。

Golang 的设计哲学鼓励使用静态类型和编译时类型检查，这有助于提高代码的可读性、可维护性和性能。所以，在实际生产中，开发人员通常倾向于使用静态类型，而将反射保留给那些确实需要在运行时处理未知类型或执行高度通用操作的场景。

```

#### 2:结构体标签

```
package main

import (
	"fmt"
	"reflect"
)

type resume struct {
	Name string `json:"name" doc:"我的名字"`
	Sex  string `json:"sex"`
}

func findTag(str interface{}) {
	t := reflect.TypeOf(str).Elem()
	for i := 0; i < t.NumField(); i++ {
		tagstring := t.Field(i).Tag.Get("json")
		fmt.Println("tag info:", tagstring)
	}
}

func main() {
	var r resume
	findTag(&r)
}

通过反射 我们可以拿到 对应的 tag 标签 

当然这个标签也不仅仅只是这个作用 他可以和json 结合使用
package main

import (
	"encoding/json"
	"fmt"
)

type Person struct {
	Name   string `json:"name"`
	Age    int    `json:"age"`
	Gender string `json:"gender,omitempty"`
}

func main() {
	p := Person{
		Name:   "Alice",
		Age:    30,
		Gender: "female",
	}

	// 将结构体转换为 JSON 字符串
	jsonData, err := json.Marshal(p)
	if err != nil {
		fmt.Println("JSON Marshal error:", err)
		return
	}

	fmt.Println(string(jsonData)) // 输出: {"name":"Alice","age":30,"gender":"female"}
}


如果我们想把之前的json在转换成结构体。
	// 再将json 转换成 结构体

	pe := Person{}
	json.Unmarshal(jsonData, &pe)
	fmt.Println(pe)



```

