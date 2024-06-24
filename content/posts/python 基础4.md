---
title: "python 作用域 元类 深浅拷贝"
date: 2022-06-27
categories:
- tranquilpeak
- features
tags:
- python
# keywords:
# - 命名空间与作用域
# - python
# - 元类
# - 深浅拷贝
thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->

### python 基础4

#### 1:命名空间与作用域

```
命名空间：----大约来说，命名空间就是一个容器，其中包含的是映射到不同对象的名称。你可能已经听说过了，Python中的一切——常量，列表，字典，函数，类，等等——都是对象。
我们可以把命名空间描述为一个Python字典结构，其中关键词代表名称，而字典值是对象本身（这也是目前Python中命名空间的实现方式）
现在比较棘手的是，我们在Python中有多个独立的命名空间，而且不同命名空间中的名称可以重复使用（只要对象是独一无二的）
每次我们调用for循环或者定义一个函数的时候，就会创建它自己的命名空间。命名空间也有不同的层次（也就是所谓的“作用域”）
使用的是LEGB规则，表示的是Local -> Enclosed -> Global -> Built-in，其中的箭头方向表示的是搜索顺序。
• Local 可能是在一个函数或者类方法内部。
• Enclosed 可能是嵌套函数内，比如说 一个函数包裹在另一个函数内部。
• Global 代表的是执行脚本自身的最高层次。
Built-in 是Python为自身保留的特殊名称。
因此，如果某个name:object映射在局部(local)命名空间中没有找到，接下来就会在闭包作用域(enclosed)进行搜索，如果闭包作用域也没有找到，Python就会到全局(global)命名空间中进行查找，最后会在内建(built-in)命名空间搜索（注：如果一个名称在所有命名空间中都没有找到，就会产生一个NameError）。

---------------------------------- 例子1--------------------------------
a_var = 'global value'
def outer():
    a_var = 'enclosed value'
    def inner():
        a_var = 'local value'
        print(a_var)
    print(a_var)
    inner()
print(a_var)
outer()

出现局部变量与全局变量时，如果有局部变量，先使用局部变量，没有时使用全局变量  一般尽量不要使用python 自带的方法关键子命名
```

#### 2: 元类

```
可以使用type函数创建元类  接受三个参数  第一个参数是类名
第二个参数是由父类名称组成的元组  第三个参数是属性的字典
AI = type("AI", (), {"hp": 100})
print(AI)
print(AI.__class__)
a1 = AI()
print(a1.__class__)
#NPC继承AI
NPC = type("NPC", (AI,), {"canMove": False})
print(NPC)
print(NPC.__bases__)
print(NPC.hp)
Python在创建NPC类时先不在内存中创建、首先查看NPC中有__metaclass__这个属性吗  可以修改类创建的顺序
如果有，Python会通过__metaclass__创建一个名字为NPC的类(对象)
如果Python没有找到__metaclass__，它会继续在AI（父类）中寻找__metaclass__属性，并尝试做和前面同样的操作。
如果Python在任何父类中都找不到__metaclass__，它就会在模块层次中去寻找__metaclass__，并尝试做同样的操作。
如果还是找不到__metaclass__,Python就会用内置的type来创建这个类对象
类是对象的抽象，对象是类
的实例化
获取类的实例，先调用__new__(静态方法)
然后通过__init__获得对象的初始化属性  __init__ 是一个实例方法
Type(object)获取对象的属性
Type(name,tuple,dict)
也就是： __new__先被调用，__init__后被调用，__new__的返回值（实例）将传递给__init__方法的第一个参数，然后__init__给这个实例设置一些参数。
```

#### 3: 深浅拷贝

```
对于  int  string   无论深浅拷贝都是指向同一块内存区域

深拷贝：数据完全不共享（复制其数据完完全全放独立的一个内存，完全拷贝，数据不共享）
浅拷贝：数据半共享（复制其数据独立内存存放，但是只拷贝成功第一层）
import copy
import types
dicta={'account':['zhangsan','lisi'],
       'type':['admin','customer'],
       'passwprd':['123','456']}
# dictb=copy.deepcopy(dicta)
dictb = dicta
dictb['account'][0]='wangwu'
dictb['type'].append('sagasg')
print(dicta)
print(dictb)
如果只是将dicta 赋值给dictb 那么改变dictb 的值 dicta 的值也会改变 因为他们共用同一个地址
所以如果想要去改变值 且原结果不变的话就要使用 深拷贝 浅拷贝只能 只能拷贝第一层
总结 ：   ====
深拷贝：对原对象的地址的拷贝，新拷贝了一份与原对象不同的地址的对象，修改对象中的任何值，都不会改变深拷贝的对象的值。
浅拷贝：对原对象的值的拷贝，地址仍是一个指针指向原对象的地址，浅拷贝或者原对象的值发生变化，那原对象和浅拷贝对象的值都会随着被改变。
```

