---
title: "GO 设计模式  介绍与实现"
date: 2022-10-31
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

### GO 设计模式  介绍与实现

#### 设计模式是什么

```
设计模式(英语 design pattern)是对面向对象设计中反复出现的问题的解决方案。这个术语是在1990年代由Erich Gamma等人从建筑设计领域引入到计算机科学中来的。这个术语的含义还存有争议。算法不是设计模式，因为算法致力于解决问题而非设计问题。设计模式通常描述了一组相互紧密作用的类与对象。设计模式提供一种讨论软件设计的公共语言，使得熟练设计者的设计经验可以被初学者和其他设计者掌握。设计模式还为软件重构提供了目标。

以上 就是设计模式的介绍。 首先一个前提就是 设计模式是以 面向对象为基础  大家都知道 go 的面向对象和 其他语言不同 
go 没有类也没有继承的概念 它更像是一种方法的组合 而不是继承 


设计模式有三大类 

1 创建型模式 2 结构型模式 行为型模式

-----------------------------创建型模式-----------------------------
单例模式（Singleton Pattern）：保证一个类仅有一个对象，并提供一个访问它的全局访问点。
工厂模式（Factory Pattern）：定义一个用于创建对象的接口，让子类决定将哪一个类实例化，FactoryMethod使一个类的实例化延迟到其子类。
抽象工厂模式（Abstract Factory Pattern）：提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们的具体类。
建造者模式（Builder Pattern）：将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。
原型模式（Prototype Pattern）:用原型实例指定创建对象的种类，并且通过拷贝这个原型来创建新的对象。
-----------------------------结构型模式-----------------------------
适配器模式（Adapter Pattern）：将一个类的接口转换成客户希望的另一个接口。Adapter模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。
桥接模式（Bridge Pattern）：将抽象部分与它的实现部分分离，使他们都可以独立地变化。
装饰模式（Decorator Pattern）：动态地给一个对象添加一些额外的职责。就扩展功能而言，Decorator模式比生成子类的方式更为灵活。
组合模式（Composite Pattern）：将对象组合成树形结构以表示“部分-整体”的层次结构。Composite 使得客户对单个对象和复合对象的使用具有一致性。
外观模式（Facade Pattern）：为子系统中的一组接口提供一个一致的接口。Façade模式定义了一个高层接口，这个接口使得这一子系统更加容易使用。
享元模式（Flyweight Pattern）：运用共享技术有效地支持大量细粒度的对象。
代理模式（Proxy Pattern）:为其他对象提供一个代理以控制对这个对象的访问。
-----------------------------行为型模式-----------------------------
模版方法模式（Template Pattern）:定义一个操作中的算法的骨架，而将一些步骤延迟到子类。TemplateMethod 使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。
命令模式（Command Pattern）：将一个请求封装为一个对象，从而使你可用不同的请求对客户进行参数化，对请求排队或记录请求日志，以及支持可取消的操作。
迭代器模式（Iterator Pattern）：提供一种方法顺序访问一个聚合对象中各个元素，而又不需暴露该对象的内部表示。
观察者模式（Observer Pattern）：定义对象间的一种一对多的依赖关系，以便当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并自动刷新。
中介者模式（Mediator Pattern）：用一个中介对象来封装一系列的对象交互。中介者使各对象不需要显示地相互引用，从而使其耦合松散，而且可以独立地改变它们之间的交互。
备忘录模式（Memento Pattern）：在不破坏封装性的前提下，捕获一个对象的内部状态，并在该对象之外保持该状态，这样以后就可以将该对象恢复到保存的状态。
解释器模式（Interpreter Pattern）：定义一个语言，定义它的文法的一种表示，并定义一个解释器，该解释器使用该表示来解释语言中的句子。
状态模式（State Pattern）:允许一个对象在其内部状态改变时改变它的行为。对象看起来似乎修改了它所属的类。
策略模式（Strategy Pattern）:定义一系列的算法，把它们一个个封装起来，并且使他们可相互替换。本模式使得算法的变化可以独立于使用它的客户。
责任链模式（Chain of Responsibility Pattern）：为解除请求的发送者和接收者之间的耦合，而使多个对象都有机会处理这个请求。将这些对象连成一条链，并沿着这条链传递该请求，直到有对象处理它。
访问者模式（Visitor Pattern）:表示一个作用于某对象结构中的各元素的操作。它使你可以在不改变各元素类别的前提下定义作用于这些元素的新操作。
```

#### 2: go 的设计模式怎么设计

```
对于面向对象软件系统的设计而言，在支持可维护性的同时，提高系统的可复用性是一个至关重要的问题，如何同时提高一个软件系统的可维护性和可复用性是面向对象设计需要解决的核心问题之一。在面向对象设计中，可维护性的复用是以设计原则为基础的。每一个原则都蕴含一些面向对象设计的思想，可以从不同的角度提升一个软件结构的设计水平。
面向对象设计原则为支持可维护性复用而诞生，这些原则蕴含在很多设计模式中，它们是从许多设计方案中总结出的指导性原则。面向对象设计原则也是我们用于评价一个设计模式的使用效果的重要指标之一。

其实原则的目的就是 高内聚 低耦合

--------------------------------------1 单一职责原则--------------------------------------

类的职责单一，对外只提供一种功能，而引起类变化的原因都应该只有一个。

--------------------------------------2 开闭原则--------------------------------------
类的改动是通过增加代码进行的，而不是修改源代码

--------------------------------------3 里式代换原则--------------------------------------

任何抽象类（interface接口 对于 go 来讲 也是接口）出现的地方都可以用他的实现类进行替换，实际就是虚拟机制，语言级别实现面向对象功能。

--------------------------------------4 依赖倒转原则--------------------------------------

依赖于抽象(接口)，不要依赖具体的实现(类)，也就是针对接口编程。

--------------------------------------5 接口隔离原则--------------------------------------

不应该强迫用户的程序依赖他们不需要的接口方法。一个接口应该只提供一种对外功能，不应该把所有操作都封装到一个接口中去。

--------------------------------------6 合成复用原则--------------------------------------

如果使用继承，会导致父类的任何变换都可能影响到子类的行为。如果使用对象组合，就降低了这种依赖关系。对于继承和组合，优先使用组合。

--------------------------------------7 迪米特法则--------------------------------------

一个对象应当对其他对象尽可能少的了解，从而降低各个对象之间的耦合，提高系统的可维护性。例如在一个程序中，各个模块之间相互调用时，通常会提供一个统一的接口来实现。这样其他模块不需要了解另外一个模块的内部实现细节，这样当一个模块内部的实现发生改变时，不会影响其他模块的使用。（黑盒原理）

```

#### 3 单一规则

```
单一职责原则
类的职责单一，对外只提供一种功能，而引起类变化的原因都应该只有一个。


package main

import "fmt"

type ClothesShop struct {}

func (cs *ClothesShop) OnShop() {
	fmt.Println("休闲的装扮")
}

type ClothesWork struct {}

func (cw *ClothesWork) OnWork() {
	fmt.Println("工作的装扮")
}

func main() {
	//工作的时候
	cw := new(ClothesWork)
	cw.OnWork()

	//shopping的时候
	cs := new(ClothesShop)
	cs.OnShop()
}


在面向对象编程的过程中，设计一个类，建议对外提供的功能单一，接口单一，影响一个类的范围就只限定在这一个接口上，一个类的一个接口具备这个类的功能含义，职责单一不复杂。


```

#### 4:开闭原则

```
开闭原则
package main

import "fmt"

//我们要写一个结构体,Banker银行业务员
type Banker struct {
}

//存款业务
func (this *Banker) Save() {
	fmt.Println( "进行了 存款业务...")
}

//转账业务
func (this *Banker) Transfer() {
	fmt.Println( "进行了 转账业务...")
}

//支付业务
func (this *Banker) Pay() {
	fmt.Println( "进行了 支付业务...")
}

func main() {
	banker := &Banker{}

	banker.Save()
	banker.Transfer()
	banker.Pay()
}

代码很简单，就是一个银行业务员，他可能拥有很多的业务，比如Save()存款、Transfer()转账、Pay()支付等。那么如果这个业务员模块只有这几个方法还好，但是随着我们的程序写的越来越复杂，银行业务员可能就要增加方法，会导致业务员模块越来越臃肿。
这样的设计会导致，当我们去给Banker添加新的业务的时候，会直接修改原有的Banker代码，那么Banker模块的功能会越来越多，出现问题的几率也就越来越大，假如此时Banker已经有99个业务了，现在我们要添加第100个业务，可能由于一次的不小心，导致之前99个业务也一起崩溃，因为所有的业务都在一个Banker类里，他们的耦合度太高，Banker的职责也不够单一，代码的维护成本随着业务的复杂正比成倍增大。



所以以上，当我们给一个系统添加一个功能的时候，不是通过修改代码，而是通过增添代码来完成，那么就是开闭原则的核心思想了。所以要想满足上面的要求，是一定需要interface来提供一层抽象的接口的。


package main

import "fmt"

//抽象的银行业务员
type AbstractBanker interface{
	DoBusi()	//抽象的处理业务接口
}

//存款的业务员
type SaveBanker struct {
	//AbstractBanker
}

func (sb *SaveBanker) DoBusi() {
	fmt.Println("进行了存款")
}

//转账的业务员
type TransferBanker struct {
	//AbstractBanker
}

func (tb *TransferBanker) DoBusi() {
	fmt.Println("进行了转账")
}

//支付的业务员
type PayBanker struct {
	//AbstractBanker
}

func (pb *PayBanker) DoBusi() {
	fmt.Println("进行了支付")
}


func main() {
	//进行存款
	sb := &SaveBanker{}
	sb.DoBusi()

	//进行转账
	tb := &TransferBanker{}
	tb.DoBusi()
	
	//进行支付
	pb := &PayBanker{}
	pb.DoBusi()

}


当然我们也可以根据AbstractBanker设计一个小框架


//实现架构层(基于抽象层进行业务封装-针对interface接口进行封装)
func BankerBusiness(banker AbstractBanker) {
	//通过接口来向下调用，(多态现象)
	banker.DoBusi()
}


func main() {
	//进行存款
	BankerBusiness(&SaveBanker{})

	//进行存款
	BankerBusiness(&TransferBanker{})

	//进行存款
	BankerBusiness(&PayBanker{})
}


最后的最后
再看开闭原则定义:
开闭原则:一个软件实体如类、模块和函数应该对扩展开放，对修改关闭。
简单的说就是在修改需求的时候，应该尽量通过扩展来实现变化，而不是通过修改已有代码来实现变化。


----------------------------------接口的意义所在----------------------------------
好了，现在interface已经基本了解，那么接口的意义最终在哪里呢，想必现在你已经有了一个初步的认知，实际上接口的最大的意义就是实现多态的思想，就是我们可以根据interface类型来设计API接口，那么这种API接口的适应能力不仅能适应当下所实现的全部模块，也适应未来实现的模块来进行调用。 调用未来可能就是接口的最大意义所在吧，这也是为什么架构师那么值钱，因为良好的架构师是可以针对interface设计一套框架，在未来许多年却依然适用。


package main

import "fmt"

// === > 奔驰汽车 <===
type Benz struct {

}

func (this *Benz) Run() {
	fmt.Println("Benz is running...")
}

// === > 宝马汽车  <===
type BMW struct {

}

func (this *BMW) Run() {
	fmt.Println("BMW is running ...")
}


//===> 司机张三  <===
type Zhang3 struct {
	//...
}

func (zhang3 *Zhang3) DriveBenZ(benz *Benz) {
	fmt.Println("zhang3 Drive Benz")
	benz.Run()
}

func (zhang3 *Zhang3) DriveBMW(bmw *BMW) {
	fmt.Println("zhang3 drive BMW")
	bmw.Run()
}

//===> 司机李四 <===
type Li4 struct {
	//...
}

func (li4 *Li4) DriveBenZ(benz *Benz) {
	fmt.Println("li4 Drive Benz")
	benz.Run()
}

func (li4 *Li4) DriveBMW(bmw *BMW) {
	fmt.Println("li4 drive BMW")
	bmw.Run()
}

func main() {
	//业务1 张3开奔驰
	benz := &Benz{}
	zhang3 := &Zhang3{}
	zhang3.DriveBenZ(benz)

	//业务2 李四开宝马
	bmw := &BMW{}
	li4 := &Li4{}
	li4.DriveBMW(bmw)
}

我们来看上面的代码和图中每个模块之间的依赖关系，实际上并没有用到任何的interface接口层的代码，显然最后我们的两个业务 张三开奔驰, 李四开宝马，程序中也都实现了。但是这种设计的问题就在于，小规模没什么问题，但是一旦程序需要扩展，比如我现在要增加一个丰田汽车 或者 司机王五， 那么模块和模块的依赖关系将成指数级递增，想蜘蛛网一样越来越难维护和捋顺。

如果我们在设计一个系统的时候，将模块分为3个层次，抽象层、实现层、业务逻辑层。那么，我们首先将抽象层的模块和接口定义出来，这里就需要了interface接口的设计，然后我们依照抽象层，依次实现每个实现层的模块，在我们写实现层代码的时候，实际上我们只需要参考对应的抽象层实现就好了，实现每个模块，也和其他的实现的模块没有关系，这样也符合了上面介绍的开闭原则。这样实现起来每个模块只依赖对象的接口，而和其他模块没关系，依赖关系单一。系统容易扩展和维护。

我们在指定业务逻辑也是一样，只需要参考抽象层的接口来业务就好了，抽象层暴露出来的接口就是我们业务层可以使用的方法，然后可以通过多态的线下，接口指针指向哪个实现模块，调用了就是具体的实现方法，这样我们业务逻辑层也是依赖抽象成编程。

我们就将这种的设计原则叫做依赖倒转原则。

来一起看一下修改的代码：

package main

import "fmt"

// ===== >   抽象层  < ========
type Car interface {
	Run()
}

type Driver interface {
	Drive(car Car)
}

// ===== >   实现层  < ========
type BenZ struct {
	//...
}

func (benz * BenZ) Run() {
	fmt.Println("Benz is running...")
}

type Bmw struct {
	//...
}

func (bmw * Bmw) Run() {
	fmt.Println("Bmw is running...")
}

type Zhang_3 struct {
	//...
}

func (zhang3 *Zhang_3) Drive(car Car) {
	fmt.Println("Zhang3 drive car")
	car.Run()
}

type Li_4 struct {
	//...
}

func (li4 *Li_4) Drive(car Car) {
	fmt.Println("li4 drive car")
	car.Run()
}


// ===== >   业务逻辑层  < ========
func main() {
	//张3 开 宝马
	var bmw Car
	bmw = &Bmw{}

	var zhang3 Driver
	zhang3 = &Zhang_3{}

	zhang3.Drive(bmw)

	//李4 开 奔驰
	var benz Car
	benz = &BenZ{}

	var li4 Driver
	li4 = &Li_4{}

	li4.Drive(benz)
}
```

#### 5 合成服用原则

```
如果使用继承，会导致父类的任何变换都可能影响到子类的行为。如果使用对象组合，就降低了这种依赖关系。对于继承和组合，优先使用组合。

package main

import "fmt"

type Cat struct {}

func (c *Cat) Eat() {
	fmt.Println("小猫吃饭")
}

//给小猫添加一个 可以睡觉的方法 （使用继承来实现）
type CatB struct {
	Cat
}

func (cb *CatB) Sleep() {
	fmt.Println("小猫睡觉")
}

//给小猫添加一个 可以睡觉的方法 （使用组合的方式）
type CatC struct {
	C *Cat
}

func (cc *CatC) Sleep() {
	fmt.Println("小猫睡觉")
}


func main() {
	//通过继承增加的功能，可以正常使用
	cb := new(CatB)
	cb.Eat()
	cb.Sleep()

	//通过组合增加的功能，可以正常使用
	cc := new(CatC)
	cc.C = new(Cat)
	cc.C.Eat()
	cc.Sleep()
}
```

#### Tips  最后总结

单一职责原则：一个类对外只提供一种功能

开闭原则：增加功能时去增加代码而不是修改代码

依赖倒转原则：模块与模块依赖抽象而不是具体实现

合成复用原则：通过组合来实现父类方法

迪米特法则：依赖第三方来实现解耦