---
title: "python assert 生成器 值传递 反射机制"
date: 2022-06-22
categories:
- tranquilpeak
- features
tags:
- python
# keywords:
# - python
# - assert
# - 生成器
# - 迭代器

thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->

### python 基础1相关

#### 1:assert 

```
很多人开发当中都会遇到的一个问题 ，在没完善一个程序之前，我们不知道程序在哪里会出错，与其让它在运行时崩溃，不如在出现错误条件时就崩溃
例如我们设置一个 会员服务时 需要打折时 其价格一定是大于0 和小于当前价格的 我们就可以设置断言 assert这样就不会出现 价格异常的情况发生。

def apply_discount(product, discount):
    price = int(product['price'] * (1 - discount))
    assert 0 <= price <= product['price'],"折扣错误"
    return price


if __name__ == "__main__":
    shoes = {
        "name": "旗舰版会员",
        "price": 1000,
    }
    price = apply_discount(shoes, 1.5)
    print(price)


```

#### 2: 生成器迭代器

```
如果一个对象拥有__iter__方法，就是一个可迭代对象

可迭代对象 ：包含了 迭代器 序列（集合 列表 元组 字符串 但他们都不是迭代器 可以通过 iter() 函数获得一个迭代器对象） 字典

迭代器 ： 一个拥有__iter__方法，和__next__方法的对象就是一个迭代器 
----------------------------------------------

iter方法是从可迭代对象中返回一个迭代器  next方法是从迭代器中获取下一条记录，如果无法获取下一条记录，则触发Stoplteration异常
迭代器是访问可迭代对象的一种方式，迭代器对象从列表的第一个元素开始访问，直到所有的元素被访问完结束，迭代器只能向前取值，不会后退

生成器是一种特殊的迭代器，生成器本生就实现了iter、next方法，只需要一个yield关键字
生成器的作用就是 节省大量的空间 ----- 
yield是一个类似return 的关键字，迭代一次遇到yield的时候就返回yield后面或者右面的值。而且下一次迭代的时候，从上一次迭代遇到的yield后面的代码开始执行.

```

#### 3: python中函数值传递与引用传递

```
值传递与引用传递简单理解 :
		值传递就是在一个参数传入到函数中后，函数中对该参数的操作不会影响函数外该参数的变量的值；而引用传递，则是参数传递进来的相当于内存地址，对该参数的操作会直接影响到外部指向其值的变量。
python中的函数传递过来的是不可变的值时（比如数字常量、元组和字符串）若传递过来的参数是可变的（如列表、字典等），则相当于引用传递。
------------------------例如---------------------
x = 10
print("xid=",id(x))
def A(x):
    print("axid=",id(x))
    return x
def B(x):
    x += x
    print("axid=", id(x))
    return x
print("ax=",A(x))
print("bx=",B(x))
print("x=",x)
print("xid=",id(x))
------------------------结果----------------------
xid= 4507184096
axid= 4507184096
ax= 10
axid= 4507184416
bx= 20
x= 10
xid= 4507184096
可以发现 无论局部的x 怎么变 都不会影响到全局的x

```

#### 4:python面向对象的反射机制

```
1: 反射的概念 
python的反射机制，核心就是利用字符串去已存在的模块中找到指定的属性或方法，找到方法后自动执行—–基于字符串的事件驱动
简单的说，在 Python 中，能够通过一个对象，找出其 type class attribute method 的能力，成为反射或者自省
具备反射能力的函数有 type() isinstance() callable() dir() getattr() 等
反射的作用
1.能够用字符串的形式去操作对象，提高了程序的灵活性和扩展性。
2.降低了耦合性，提升了代码的健壮性和自适应能力。
3.这种形式可以应对任何类的对象。
```

#### 

