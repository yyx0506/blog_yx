

---
title: "Golang 设计模式之创建型模式"
date: 2022-11-29
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

###Golang 设计模式之结构型模式

| **模式名称**                          | **模式名称**    | **作用**                                                     |
| ------------------------------------- | --------------- | ------------------------------------------------------------ |
| **结构型模式**Structural Pattern（7） | 适配器模式★★★★☆ | 将一个类的接口转换成客户希望的另外一个接口。使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。 |
|                                       | 桥接模式★★★☆☆   | 将抽象部分与实际部分分离，使它们都可以独立的变化。           |
|                                       | 组合模式★★☆☆☆   | 将对象组合成树形结构以表示“部分--整体”的层次结构。使得用户对单个对象和组合对象的使用具有一致性。 |
|                                       | 装饰模式★★★☆☆   | 动态的给一个对象添加一些额外的职责。就增加功能来说，此模式比生成子类更为灵活。 |
|                                       | 外观模式★★★★★   | 为子系统中的一组接口提供一个一致的界面，此模式定义了一个高层接口，这个接口使得这一子系统更加容易使用。 |
|                                       | 享元模式★☆☆☆☆   | 以共享的方式高效的支持大量的细粒度的对象。                   |
|                                       | 代理模式★★★★☆   | 为其他对象提供一种代理以控制对这个对象的访问。               |

本篇博客我们 只讲 代理模式 装饰模式 适配器模式 外观模式

#### 1 代理模式

```
Proxy模式又叫做代理模式，是构造型的设计模式之一，它可以为其他对象提供一种代理（Proxy）以控制对这个对象的访问。	
	所谓代理，是指具有与代理元（被代理的对象）具有相同的接口的类，客户端必须通过代理与被代理的目标类交互，而代理一般在交互的过程中（交互前后），进行某些特别的处理。
	
	这里假设有一个“自己”的角色，正在玩一款网络游戏。称这个网络游戏就是代理模式的“Subject”，表示要做一件事的目标或者对象事件主题。
（1）“自己”有一个给游戏角色升级的需求或者任务，当然“自己”可以独自完成游戏任务的升级。
（2）或者“自己”也可以邀请以为更加擅长游戏的“游戏代练”来完成升级这件事，这个代练就是“Proxy”代理。
（3）“游戏代练”不仅能够完成升级的任务需求，还可以额外做一些附加的能力。比如打到一些好的游戏装备、加入公会等等周边收益。
所以代理的出现实则是为了能够覆盖“自己”的原本的需求，且可以额外做其他功能，这种额外创建的类是不影响已有的“自己”和“网络游戏”的的关系。是额外添加，在设计模式原则上，是符合“开闭原则”思想。那么当需要给“自己”增加额外功能的时候，又不想改变自己，那么就选择邀请一位”代理”来完成吧。
```

代理模式的标准类图如下：

![](/img/design8.png)

**subject（抽象主题角色）**：真实主题与代理主题的共同接口。

​	**RealSubject（真实主题角色）**：定义了代理角色所代表的真实对象。 

​	**Proxy（代理主题角色）：**含有对真实主题角色的引用，代理角色通常在将客户端调用传递给真是主题对象之前或者之后执行某些操作，而不是单纯返回真实的对象。

代理模式案例实现 如图所示

![](/img/design9.png)

这里以一个购物作为一个主题任务，这是一个抽象的任务。具体的购物主题分别包括“韩国购物”、“美国购物”、“非洲购物”等。可以这些都是“自己”去完成主题，那么如果希望不仅完成购物，还要做真假辨别、海关安检等，同样依然能够完成自己本身的具体购物主题，那么则可以创建一个新的代理来完成这件事。代理需要将被代理的主题关联到本类中，去重新实现Buy()方法，在Buy()方法中，调用被调用的Buy()，在额外完成辨别真伪和海关安检两个任务动作，具体的代码实现如下：

```GO
package main

import "fmt"

type Goods struct {
	Kind string   //商品种类
	Fact bool	  //商品真伪
}

// =========== 抽象层 ===========
//抽象的购物主题Subject
type Shopping interface {
	Buy(goods *Goods) //某任务
}


// =========== 实现层 ===========
//具体的购物主题， 实现了shopping， 去韩国购物
type KoreaShopping struct {}

func (ks *KoreaShopping) Buy(goods *Goods) {
	fmt.Println("去韩国进行了购物, 买了 ", goods.Kind)
}


//具体的购物主题， 实现了shopping， 去美国购物
type AmericanShopping struct {}

func (as *AmericanShopping) Buy(goods *Goods) {
	fmt.Println("去美国进行了购物, 买了 ", goods.Kind)
}

//具体的购物主题， 实现了shopping， 去非洲购物
type AfrikaShopping struct {}

func (as *AfrikaShopping) Buy(goods *Goods) {
	fmt.Println("去非洲进行了购物, 买了 ", goods.Kind)
}


//海外的代理
type OverseasProxy struct {
	shopping Shopping //代理某个主题，这里是抽象类型
}

func (op *OverseasProxy) Buy(goods *Goods) {
	// 1. 先验货
	if (op.distinguish(goods) == true) {
		//2. 进行购买
		op.shopping.Buy(goods) //调用原被代理的具体主题任务
		//3 海关安检
		op.check(goods)
	}
}

//创建一个代理,并且配置关联被代理的主题
func NewProxy(shopping Shopping) Shopping {
	return &OverseasProxy{shopping}
}

//验货流程
func (op *OverseasProxy) distinguish(goods *Goods) bool {
	fmt.Println("对[", goods.Kind,"]进行了辨别真伪.")
	if (goods.Fact == false) {
		fmt.Println("发现假货",goods.Kind,", 不应该购买。")
	}
	return goods.Fact
}

//安检流程
func (op *OverseasProxy) check(goods *Goods) {
	fmt.Println("对[",goods.Kind,"] 进行了海关检查， 成功的带回祖国")
}


func main() {
	g1 := Goods{
		Kind: "韩国面膜",
		Fact: true,
	}

	g2 := Goods{
		Kind: "CET4证书",
		Fact: false,
	}

	//如果不使用代理来完成从韩国购买任务
	var shopping Shopping
	shopping = new(KoreaShopping) //具体的购买主题

	//1-先验货
	if g1.Fact == true {
		fmt.Println("对[", g1.Kind,"]进行了辨别真伪.")
		//2-去韩国购买
		shopping.Buy(&g1)
		//3-海关安检
		fmt.Println("对[",g1.Kind,"] 进行了海关检查， 成功的带回祖国")
	}

	fmt.Println("---------------以下是 使用 代理模式-------")
	var overseasProxy Shopping
	overseasProxy = NewProxy(shopping)
	overseasProxy.Buy(&g1)
	overseasProxy.Buy(&g2)
}

```

运行结果如下：

```GO
对[ 韩国面膜 ]进行了辨别真伪.
去韩国进行了购物, 买了  韩国面膜
对[ 韩国面膜 ] 进行了海关检查， 成功的带回祖国
---------------以下是 使用 代理模式-------
对[ 韩国面膜 ]进行了辨别真伪.
去韩国进行了购物, 买了  韩国面膜
对[ 韩国面膜 ] 进行了海关检查， 成功的带回祖国
对[ CET4证书 ]进行了辨别真伪.
发现假货 CET4证书 , 不应该购买。
```

代理模式纯理解案例代码如下

```GO
package main

import "fmt"

//抽象主题
type BeautyWoman interface {
	//对男人抛媚眼
	MakeEyesWithMan()
	//和男人浪漫的约会
	HappyWithMan()
}


//具体主题
type PanJinLian struct {}

//对男人抛媚眼
func (p *PanJinLian) MakeEyesWithMan() {
	fmt.Println("潘金莲对本官抛了个媚眼")
}

//和男人浪漫的约会
func (p *PanJinLian) HappyWithMan() {
	fmt.Println("潘金莲和本官共度了浪漫的约会。")
}

//代理中介人, 王婆
type WangPo struct {
	woman BeautyWoman
}

func NewProxy(woman BeautyWoman) BeautyWoman {
	return &WangPo{woman}
}

//对男人抛媚眼
func (p *WangPo) MakeEyesWithMan() {
	p.woman.MakeEyesWithMan()
}

//和男人浪漫的约会
func (p *WangPo) HappyWithMan() {
	p.woman.HappyWithMan()
}


//西门大官人
func main() {
	//大官人想找金莲，让王婆来安排
	wangpo := NewProxy(new(PanJinLian))
	//王婆命令潘金莲抛媚眼
	wangpo.MakeEyesWithMan()
	//王婆命令潘金莲和西门庆约会
	wangpo.HappyWithMan()
}

```

结果如下：

```
潘金莲对本官抛了个媚眼
潘金莲和本官共度了浪漫的约会。
```



```
总结
代理模式的优缺点
优点：
    (1) 能够协调调用者和被调用者，在一定程度上降低了系统的耦合度。
    (2) 客户端可以针对抽象主题角色进行编程，增加和更换代理类无须修改源代码，符合开闭原则，系统具有较好的灵活性和可扩展性。
缺点：
    (1) 代理实现较为复杂。
```

####  2 装饰模式

装饰模式(Decorator Pattern)：动态地给一个对象增加一些额外的职责，就增加对象功能来说，装饰模式比生成子类实现更为灵活。装饰模式是一种对象结构型模式。 如图所示 ：

![](/img/design10.png)

以上图为例，一开始有个手机（裸机Phone类），如果需要不断的为这个Phone增添某个功能从而变成一个新功能的Phone，就需要一个装饰器的类，来动态的给一个类额外添加一个指定的功能，而生成另一个类，但原先的类又没有改变，不影响原有系统的稳定。

在装饰器模式中，“裸机”、“有贴膜的手机”、“有手机壳的手机”、“有手机壳&贴膜的手机”都是一个构件。

“贴膜装饰器”、“手机壳装饰器”是装饰器也是一个构件。



装饰模式中的角色和职责:

**Component（抽象构件）**：它是具体构件和抽象装饰类的共同父类，声明了在具体构件中实现的业务方法，它的引入可以使客户端以一致的方式处理未被装饰的对象以及装饰之后的对象，实现客户端的透明操作。

**ConcreteComponent（具体构件）**：它是抽象构件类的子类，用于定义具体的构件对象，实现了在抽象构件中声明的方法，装饰器可以给它增加额外的职责（方法）。

其标准的类图如下所示

![](/img/design11.png)

接下来按照上述手机案例，结合装饰器的设计模式特点，得到对应案例的类图，如下：

![](/img/design12.png)

实现的代码如下：

```GO
package main

import "fmt"

// ---------- 抽象层 ----------
//抽象的构件
type Phone interface {
	Show() //构件的功能
}

//装饰器基础类（该类本应该为interface，但是Golang interface语法不可以有成员属性）
type Decorator struct {
	phone Phone
}

func (d *Decorator) Show() {}


// ----------- 实现层 -----------
// 具体的构件
type HuaWei struct {}

func (hw *HuaWei) Show() {
	fmt.Println("秀出了HuaWei手机")
}

type XiaoMi struct{}

func (xm *XiaoMi) Show() {
	fmt.Println("秀出了XiaoMi手机")
}

// 具体的装饰器类
type MoDecorator struct {
	Decorator   //继承基础装饰器类(主要继承Phone成员属性)
}

func (md *MoDecorator) Show() {
	md.phone.Show() //调用被装饰构件的原方法
	fmt.Println("贴膜的手机") //装饰额外的方法
}

func NewMoDecorator(phone Phone) Phone {
	return &MoDecorator{Decorator{phone}}
}

type KeDecorator struct {
	Decorator   //继承基础装饰器类(主要继承Phone成员属性)
}

func (kd *KeDecorator) Show() {
	kd.phone.Show()
	fmt.Println("手机壳的手机") //装饰额外的方法
}

func NewKeDecorator(phone Phone) Phone {
	return &KeDecorator{Decorator{phone}}
}


// ------------ 业务逻辑层 ---------
func main() {
	var huawei Phone
	huawei = new(HuaWei)
	huawei.Show()	 //调用原构件方法

	fmt.Println("---------")
	//用贴膜装饰器装饰，得到新功能构件
	var moHuawei Phone
	moHuawei = NewMoDecorator(huawei) //通过HueWei ---> MoHuaWei
	moHuawei.Show() //调用装饰后新构件的方法

	fmt.Println("---------")
	var keHuawei Phone
	keHuawei = NewKeDecorator(huawei) //通过HueWei ---> KeHuaWei
	keHuawei.Show()

	fmt.Println("---------")
	var keMoHuaWei Phone
	keMoHuaWei = NewMoDecorator(keHuawei) //通过KeHuaWei ---> KeMoHuaWei
	keMoHuaWei.Show()
}

```

运行结果如下：

```
秀出了HuaWei手机
---------
秀出了HuaWei手机
贴膜的手机
---------
秀出了HuaWei手机
手机壳的手机
---------
秀出了HuaWei手机
手机壳的手机
贴膜的手机
```



```
总结
优点：
    (1) 对于扩展一个对象的功能，装饰模式比继承更加灵活性，不会导致类的个数急剧增加。
    (2) 可以通过一种动态的方式来扩展一个对象的功能，从而实现不同的行为。
    (3) 可以对一个对象进行多次装饰。
    (4) 具体构件类与具体装饰类可以独立变化，用户可以根据需要增加新的具体构件类和具体装饰类，原有类库代码无须改变，符合“开闭原则”。
缺点：
    (1) 使用装饰模式进行系统设计时将产生很多小对象，大量小对象的产生势必会占用更多的系统资源，影响程序的性能。
    (2) 装饰模式提供了一种比继承更加灵活机动的解决方案，但同时也意味着比继承更加易于出错，排错也很困难，对于多次装饰的对象，调试时寻找错误可能需要逐级排查，较为繁琐。

适用场景
    (1) 动态、透明的方式给单个对象添加职责。
    (2) 当不能采用继承的方式对系统进行扩展或者采用继承不利于系统扩展和维护时可以使用装饰模式。
    装饰器模式关注于在一个对象上动态的添加方法，然而代理模式关注于控制对对象的访问。换句话说，用代理模式，代理类（proxy class）可以对它的客户隐藏一个对象的具体信息。因此，当使用代理模式的时候，我们常常在一个代理类中创建一个对象的实例。并且，当我们使用装饰器模式的时候，我们通常的做法是将原始对象作为一个参数传给装饰者的构造器。

```

#### 3 适配器模式

适配器模式的标准类图如下：

![](/img/design13.png)

**Target（目标抽象类）：**目标抽象类定义客户所需接口，可以是一个抽象类或接口，也可以是具体类。

**Adapter（适配器类）：**适配器可以调用另一个接口，作为一个转换器，对Adaptee和Target进行适配，适配器类是适配器模式的核心，在对象适配器中，它通过继承Target并关联一个Adaptee对象使二者产生联系。

**Adaptee（适配者类）：**适配者即被适配的角色，它定义了一个已经存在的接口，这个接口需要适配，适配者类一般是一个具体类，包含了客户希望使用的业务方法，在某些情况下可能没有适配者类的源代码。

根据对象适配器模式结构图，在对象适配器中，客户端需要调用request()方法，而适配者类Adaptee没有该方法，但是它所提供的specificRequest()方法却是客户端所需要的。为了使客户端能够使用适配者类，需要提供一个包装类Adapter，即适配器类。这个包装类包装了一个适配者的实例，从而将客户端与适配者衔接起来，在适配器的request()方法中调用适配者的specificRequest()方法。因为适配器类与适配者类是关联关系（也可称之为委派关系），所以这种适配器模式称为对象适配器模式。



上述的例子，来重新设计类图如下：

![](/img/design14.png)

实现代码如下

```GO
package main

import "fmt"

//适配的目标
type V5 interface {
	Use5V()
}

//业务类，依赖V5接口
type Phone struct {
	v V5
}

func NewPhone(v V5) *Phone {
	return &Phone{v}
}

func (p *Phone) Charge() {
	fmt.Println("Phone进行充电...")
	p.v.Use5V()
}


//被适配的角色，适配者
type V220 struct {}

func (v *V220) Use220V() {
	fmt.Println("使用220V的电压")
}

//电源适配器
type Adapter struct {
	v220 *V220
}

func (a *Adapter) Use5V() {
	fmt.Println("使用适配器进行充电")

	//调用适配者的方法
	a.v220.Use220V()
}

func NewAdapter(v220 *V220) *Adapter {
	return &Adapter{v220}
}



// ------- 业务逻辑层 -------
func main() {
	iphone := NewPhone(NewAdapter(new(V220)))

	iphone.Charge()
}
```

运行结果如下：

```GO
Phone进行充电...
使用适配器进行充电
使用220V的电压
```



```
总结
优点：
    (1) 将目标类和适配者类解耦，通过引入一个适配器类来重用现有的适配者类，无须修改原有结构。
    (2) 增加了类的透明性和复用性，将具体的业务实现过程封装在适配者类中，对于客户端类而言是透明的，而且提高了适配者的复用性，同一个适配者类可以在多个不同的系统中复用。
    (3) 灵活性和扩展性都非常好，可以很方便地更换适配器，也可以在不修改原有代码的基础上增加新的适配器类，完全符合“开闭原则”。
缺点:
		适配器中置换适配者类的某些方法比较麻烦。
适应场景
    (1) 系统需要使用一些现有的类，而这些类的接口（如方法名）不符合系统的需要，甚至没有这些类的源代码。
    (2) 想创建一个可以重复使用的类，用于与一些彼此之间没有太大关联的一些类，包括一些可能在将来引进的类一起工作。
```



#### 4 外观模式

根据迪米特法则，如果两个类不必彼此直接通信，那么这两个类就不应当发生直接的相互作用。

​	Facade模式也叫外观模式，是由GoF提出的23种设计模式中的一种。Facade模式为一组具有类似功能的类群，比如类库，子系统等等，提供一个一致的简单的界面。这个一致的简单的界面被称作facade。

外观模式的标准类图如下：

![](/img/design15.png)

**Façade(外观角色)：**为调用方, 定义简单的调用接口。

**SubSystem(子系统角色)：**功能提供者。指提供功能的类群（模块或子系统）。



外观模式相对简单一些，代码实现如下：

```go
package main

import "fmt"

type SubSystemA struct {}

func (sa *SubSystemA) MethodA() {
	fmt.Println("子系统方法A")
}

type SubSystemB struct {}

func (sb *SubSystemB) MethodB() {
	fmt.Println("子系统方法B")
}

type SubSystemC struct {}

func (sc *SubSystemC) MethodC() {
	fmt.Println("子系统方法C")
}

type SubSystemD struct {}

func (sd *SubSystemD) MethodD() {
	fmt.Println("子系统方法D")
}

//外观模式，提供了一个外观类， 简化成一个简单的接口供使用
type Facade struct {
	a *SubSystemA
	b *SubSystemB
	c *SubSystemC
	d *SubSystemD
}

func (f *Facade) MethodOne() {
	f.a.MethodA()
	f.b.MethodB()
}


func (f *Facade) MethodTwo() {
	f.c.MethodC()
	f.d.MethodD()
}

func main() {
	//如果不用外观模式实现MethodA() 和 MethodB()
	sa := new(SubSystemA)
	sa.MethodA()
	sb := new(SubSystemB)
	sb.MethodB()

	fmt.Println("-----------")
	//使用外观模式
	f := Facade{
		a: new(SubSystemA),
		b: new(SubSystemB),
		c: new(SubSystemC),
		d: new(SubSystemD),
	}

	//调用外观包裹方法
	f.MethodOne()
}
```

结果如下

```
子系统方法A
子系统方法B
-----------
子系统方法A
子系统方法B
```

外观模式的案例 如图所示

![](/img/design16.png)

代码如下：

```GO
package main

import "fmt"

//电视机
type TV struct {}

func (t *TV) On() {
	fmt.Println("打开 电视机")
}

func (t *TV) Off() {
	fmt.Println("关闭 电视机")
}


//电视机
type VoiceBox struct {}

func (v *VoiceBox) On() {
	fmt.Println("打开 音箱")
}

func (v *VoiceBox) Off() {
	fmt.Println("关闭 音箱")
}

//灯光
type Light struct {}

func (l *Light) On() {
	fmt.Println("打开 灯光")
}

func (l *Light) Off() {
	fmt.Println("关闭 灯光")
}


//游戏机
type Xbox struct {}

func (x *Xbox) On() {
	fmt.Println("打开 游戏机")
}

func (x *Xbox) Off() {
	fmt.Println("关闭 游戏机")
}


//麦克风
type MicroPhone struct {}

func (m *MicroPhone) On() {
	fmt.Println("打开 麦克风")
}

func (m *MicroPhone) Off() {
	fmt.Println("关闭 麦克风")
}

//投影仪
type Projector struct {}

func (p *Projector) On() {
	fmt.Println("打开 投影仪")
}

func (p *Projector) Off() {
	fmt.Println("关闭 投影仪")
}


//家庭影院(外观)
type HomePlayerFacade struct {
	tv TV
	vb VoiceBox
	light Light
	xbox Xbox
	mp MicroPhone
	pro Projector
}


//KTV模式
func (hp *HomePlayerFacade) DoKTV() {
	fmt.Println("家庭影院进入KTV模式")
	hp.tv.On()
	hp.pro.On()
	hp.mp.On()
	hp.light.Off()
	hp.vb.On()
}

//游戏模式
func (hp *HomePlayerFacade) DoGame() {
	fmt.Println("家庭影院进入Game模式")
	hp.tv.On()
	hp.light.On()
	hp.xbox.On()
}

func main() {
	homePlayer := new(HomePlayerFacade)

	homePlayer.DoKTV()

	fmt.Println("------------")

	homePlayer.DoGame()
}

```

结果如下

```
家庭影院进入KTV模式
打开 电视机
打开 投影仪
打开 麦克风
关闭 灯光
打开 音箱
------------
家庭影院进入Game模式
打开 电视机
打开 灯光
打开 游戏机
```



```
总结
优点：
    (1) 它对客户端屏蔽了子系统组件，减少了客户端所需处理的对象数目，并使得子系统使用起来更加容易。通过引入外观模式，客户端代码将变得很简单，与之关联的对象也很少。
    (2) 它实现了子系统与客户端之间的松耦合关系，这使得子系统的变化不会影响到调用它的客户端，只需要调整外观类即可。
    (3) 一个子系统的修改对其他子系统没有任何影响。
缺点：
    (1) 不能很好地限制客户端直接使用子系统类，如果对客户端访问子系统类做太多的限制则减少了可变性和灵活 性。
    (2) 如果设计不当，增加新的子系统可能需要修改外观类的源代码，违背了开闭原则。

适用场景
    (1) 复杂系统需要简单入口使用。
    (2) 客户端程序与多个子系统之间存在很大的依赖性。
    (3) 在层次化结构中，可以使用外观模式定义系统中每一层的入口，层与层之间不直接产生联系，而通过外观类建立联系，降低层之间的耦合度。
```

