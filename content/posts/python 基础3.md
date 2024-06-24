---
title: "python 垃圾回收 "
date: 2022-06-26
categories:
- tranquilpeak
- features
tags:
- python
# keywords:
# - 垃圾回收机制
# - python
# - 面向对象
# - all
thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->

### python 基础3

#### 1:垃圾回收机制

```
python 垃圾回收机制主要分为 1 引用计数 2 标记清除 3 分代回收
------------------------------引用计数------------------------------
引用计数+1的情况
1、对象被创建时，例如a="hello zzy"
2、对象被copy引用时，例如 b=a，此时a引用计数+1
3、对象被作为参数，传入到一个函数中时
4、对象作为一个子元素，存储到容器中时，例如 list=[a,b]
引用计数-1的情况
1、对象别名被显示销毁，例如 del a
2、对象引用被赋予新的对象，例如b=c，此时b引用计数-1（对照引用计数+1的情况下的第二点来看）
3、一个函数离开他的作用域，例如函数执行完成，它的引用参数的引用计数-1
4、对象所在容器被销毁，或者从容器中删除。
------------------------------标记清除------------------------------
Garbage collection 也就是我们常说的gc模块 
它分为两个阶段：第一阶段是标记阶段，GC会把所有的『活动对象』打上标记，第二阶段是把那些没有标记的对象『非活动对象』进行回收
为什么要标记清除：为了解决引用计数器循环引用的不足，循环引用可能导致内存泄漏
------------------------------分代回收------------------------------
分代回收是一种以空间换时间的操作方式，Python将内存根据对象的存活时间划分为不同的集合，每个集合称为一个代，Python将内存分为了3“代”，分别为年轻代（第0代）、中年代（第1代）、老年代（第2代），他们对应的是3个链表，它们的垃圾收集频率与对象的存活时间的增大而减小。

在python中，维护了一个refchain的双向循环环状链表，这个链表中存储程序创建的所有对象，每种类型的对象中都有一个0b_refcnt引用计数器的值，默认为1，当引用计数器变为0时会进行垃圾回收(对象销毁，refchain中移除)

但是，在python 中，对于那些可以有多个元素组成的对象可能会存在循环引用的问题，为了解决这个问题，python又引入了标记清除和分代回收，在其内部维护了四个链表 refchain  
```

#### 2: all 的使用

```
在一个 py 文件 如果有方法不希望其他的文件能够倒入，只在此文件中使用的话 可以使用 __all__ 方法进行限制
例如
__all__ = ["text1"]
则 此文件的 其他方法无法导入 __all__也是对于模块公开接口的一种约定，比起下划线，__all__提供了暴露接口用的“白名单”

__all__变量只在以from 模块名 import *形式导入模块时起作用，而以其他形式，如import 模块名、from 模块名 import 成员时都不起作用。
代码中当然是不提倡用 from xxx import * 的写法的，一般只用在临时代码如console调试中，这种时候如果没有定义__all__，会将模块中非下划线开头的所有成员都导入当前命名空间中，可能弄脏当前命名空间。
__all__应该是list 类型的。
不应该动态生成__all__，比如使用列表解析式。
__all__的位置：按照 PEP8 建议的风格，__all__应该写在所有 import 语句下面，和函数、常量等模块成员定义的上面。
```

#### 3: 面向对象的反射机制

```
反射的概念 ：
python的反射机制，核心就是利用字符串去已存在的模块中找到指定的属性或方法，找到方法后自动执行—–基于字符串的事件驱动
什么是反射 ：
指的是程序运行过程中可以动态（在程序运行时）获取对象的信息
为何要用反射：
Python是一门动态语言，只有在程序运行时才能知道数据的类型及对象包含的属性和方法，比如：
def func(obj):
    return obj.x#当调用传入一个obj对象时，首先要判断该对象是否包含‘x’这个属性
 
func(10)# 若随意传入一个值时则会报错
 
例如 下面的代码 就是正确的
class ABC(object):
    x = 2
    def xqq(self):
        print("cc")

def func(obj):
    if 'x' not in obj.__dict__:#应当判断是否含有'x'这个属性
        return None
    return obj.x
ss = func(ABC)
print(ss)

四个 内置函数
hasattr()# 判断对象是否存在
print(hasattr(obj,'name'))
 
getattr()# 获取属性
print(getattr(obj,'name'))
 
setattr()# 为属性赋值
print(setattr(obj,'name','egon'))
print(obj.name)
 
delattr()# 删除属性
delattr(obj,'name')

-------------------------------------------使用场景-----------------------------------------
一个简答的例子来说明
class WebSite:
    def register(self):
        print("欢迎来到注册页面")
    
    def login(self):
        print("欢迎来到登录页面")
    
    def home(self):
        print("欢迎进入主页")
        
    def about(self):
        print("关于我们")
        
    def error(self):
        print("404 No Found!")

page = WebSite()        
while True:
    choose = input("请输入你要进入的页面>>>")
    if choose == 'register':
        page.register()
    elif choose == 'login':
        page.login()
    elif choose == 'home':
        page.home()
    elif choose == 'about':
        page.about()
    else:
        page.error()
        
我们改写以下下面的选择分支
page = WebSite()        
while True:
    choose = input("请输入你要进入的页面>>>")
    # 反射机制实现上述功能，优化代码结构
    if hasattr(page,choose):
        f = getattr(page,choose)
    else:
        page.error()

```

