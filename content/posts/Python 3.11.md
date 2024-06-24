
---
title: "Python 3.11 特性1"
date: 2022-12-13
categories:
- tranquilpeak
- features
tags:
- python
# keywords:
# - python

thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->



### Python 3.11

前言：

​		Python官方在2020年1月1日结束了对Python2的维护，这也意味着Python2已经成为历史，真正进入了Python3时代。自从进入Python3版本以来，官方已经发布了众多修改版本分支，现在最新的正式版本就是前不久刚发布的Python3.11，这一版本历经17个月的开发，现在公开可用，据说对用户更友好。

###  更新信息：

​		首先 是 ：[官方文档]("https://docs.python.org/3.11/whatsnew/3.11.html")

​		1：速度更快 ：首先第一点，也是最重要的，就是它更快了，官方说法是Python 3.11比Python 3.10快10-60%。

​		2：更加人性化的报错 [官方文档]("https://docs.python.org/3.11/whatsnew/3.11.html#whatsnew311-pep657")

​			  更加人性化的报错，与之前显示在某一行的报错不同，这个版本的报错会显示具体原因，画出具体位置。

### 增强回溯中的错误位置。

当我们打印tracebacks，解释器现在指出特定表达式引起的错误，而不是仅仅某一行，比如：

```python3
Traceback (most recent call last):
  File "distance.py", line 11, in <module>
    print(manhattan_distance(p1, p2))
          ^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "distance.py", line 6, in manhattan_distance
    return abs(point_1.x - point_2.x) + abs(point_1.y - point_2.y)
                           ^^^^^^^^^
AttributeError: 'NoneType' object has no attribute 'x'
```

以前版本的解释器会只指向这一行，使得哪个对象是None变得模糊不清。在处理深度嵌套的字典对象和多个函数调用时，这些增强的错误也会有帮助。

### 异常情况可以用注释来补充

`add_note()`方法被添加到`BaseException`中。它可以用来丰富异常的上下文信息，这些信息在异常发生的时候是不可用的。添加的注释会出现在默认的跟踪回溯中。

```python3
>>> try:
...     raise TypeError('bad type')
... except Exception as e:
...     e.add_note('Add some information')
...     raise
...
Traceback (most recent call last):
  File "<stdin>", line 2, in <module>
TypeError: bad type
Add some information
>>>
```

## 新的类型标注

### 将单个TypedDict项目标记为需要或不需要。

`Required` 和 `NotRequired` 提供了一种直接的方式来标记 `TypedDict` 中的各个项目是否必须存在。以前，这只能通过继承来实现。

除非设置了` total=False` 参数，否则默认情况下字段仍然是必需的。例如，下面指定了一个有一个必须的和一个非必须的键的 `dictionary`。

```python3
class Movie(TypedDict):
   title: str
   year: NotRequired[int]

m1: Movie = {"title": "Black Panther", "year": 2018}  # ok
m2: Movie = {"title": "Star Wars"}  # ok (year is not required)
m3: Movie = {"year": 2022}  # error (missing required field title)
```

等价于

```python3
class Movie(TypedDict, total=False):
   title: Required[str]
   year: int
```

python3.11 官方说比python3.10平均快25%，主要来自

**快速启动**，在Python 3.11中，解释器的启动速度现在提高了10-15%。这对使用Python的短时运行的程序有很大影响。

**快速运行**，更便捷的、懒惰的Python框架，优化内嵌的Python函数调用，专业化自适应的解释器。

其实，我感觉Python最大的优势就是开发速度快，可以快速实现自己的想法，学习成本也很低，优化运行速度，可能对大多数都无法感知到。

确实要把「类型标注」的规范加强一点，动态语言有个劣势就是代码写多了，都不知道变量是啥数据类型的，这让别人接手都是抓狂的。

### Collections 容器数据类型

[官方文档]("https://docs.python.org/zh-cn/3/library/collections.html")

```
这个模块之前的文档中介绍过 
在3.11中也做了更新
----------------------新版功能----------------------
一个 ChainMap 类是为了将多个映射快速的链接到一起，这样它们就可以作为一个单元处理。它通常比创建一个新字典和多次调用 update() 要快很多。
这个类可以用于模拟嵌套作用域，并且在模版化的时候比较有用。

class collections.ChainMap(*maps)
一个 ChainMap 将多个字典或者其他映射组合在一起，创建一个单独的可更新的视图。 如果没有 maps 被指定，就提供一个默认的空字典，这样一个新链至少有一个映射。

底层映射被存储在一个列表中。这个列表是公开的，可以通过 maps 属性存取和更新。没有其他的状态。

搜索查询底层映射，直到一个键被找到。不同的是，写，更新和删除只操作第一个映射。

一个 ChainMap 通过引用合并底层映射。 所以，如果一个底层映射更新了，这些更改会反映到 ChainMap 。

支持所有常用字典方法。另外还有一个 maps 属性(attribute)，一个创建子上下文的方法(method)， 一个存取它们首个映射的属性(property):

maps
一个可以更新的映射列表。这个列表是按照第一次搜索到最后一次搜索的顺序组织的。它是仅有的存储状态，可以被修改。列表最少包含一个映射。

new_child(m=None, **kwargs)
返回一个新的 ChainMap，其中包含一个新的映射，后面跟随当前实例中的所有映射。 如果指定了 m，它会成为新的映射加在映射列表的前面；如果未指定，则会使用一个空字典，因此调用 d.new_child() 就等价于 ChainMap({}, *d.maps)。 如果指定了任何关键字参数，它们会更新所传入的映射或新的空字典。 此方法被用于创建子上下文，它可在不改变任何上级映射的情况下被更新。

在 3.4 版更改: 添加了 m 可选参数。

在 3.10 版更改: 增加了对关键字参数的支持。
parents
属性返回一个新的 ChainMap 包含所有的当前实例的映射，除了第一个。这样可以在搜索的时候跳过第一个映射。 使用的场景类似在 nested scopes 嵌套作用域中使用 nonlocal 关键词。用例也可以类比内建函数 super() 。一个 d.parents 的引用等价于 ChainMap(*d.maps[1:]) 。

注意，一个 ChainMap() 的迭代顺序是通过从后往前扫描所有映射来确定的:
baseline = {'music': 'bach', 'art': 'rembrandt'}
adjustments = {'art': 'van gogh', 'opera': 'carmen'}
print(list(ChainMap(adjustments, baseline)))
['music', 'art', 'opera']
这给出了与 dict.update() 调用序列相同的顺序，从最后一个映射开始:
combined = baseline.copy()
combined.update(adjustments)
print(list(combined))
```

