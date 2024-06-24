---
title: "python list set lambda 高级用法"
date: 2022-06-29
categories:
- tranquilpeak
- features
tags:
- python
# keywords:
# - collections
# - python
# - calendar
thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->
#### 1: sort sorted

```
sort()与sorted()的不同在于，sort是在原位重新排列列表，而sorted()是产生一个新的列表。
a = [1,2,3,11,7,13]
a.sort(reverse=True)
print(a)  默认 是 reverse False
a = [1, 2, 3, 11, 7, 13]
b = sorted(a,reverse=True)
print(b)
sort 是应用在 list 上的方法，sorted 可以对所有可迭代的对象进行排序操作。 例如
Demo_dict = {
"aaa":1,
"bbb":11,
"ccc":21,
"ddd":18
}
new_dict = sorted([[key, int(value)] for key, value in Demo_dict.items()], key=lambda x: x[1], reverse=True)
print(new_dict)
```

#### 2: list set 

```
list 是列表 set 是集合 都是python 的数据结构之一，他们有很多的操作是可以非常有用的
1 : difference()
用于查找两个集合之间的差，该方法以该集合(set1)调用，另一个集合(set2)作为参数传递，并且它返回set2中不存在的元素集。
例如  注意 声明一个set 时 只接受一个参数是可迭代对象： 
 seta = set('1467')
 setb = set('2467')
 print(seta.difference(setb)) 有一点要注意 set是无序的如果后续要进行对应位置的操作还需要进行其他的操作
 ---- 结果 {'1'}
2 : intersection()
intersection() 方法用于返回两个或更多集合中都包含的元素，即交集。
 seta = set('1467')
 setb = set('2467')
 print(seta.intersection(setb))
 ---- 结果 注意是无序的 {'6', '7', '4'}
```

#### 3:lambda

```
lambda 作为一个匿名函数 在任何语言都有类似的功能 在python中也不例外
lambda 函数的语法只包含一个语句，表现形式如下：  
lambda [arg1 [,arg2,.....argn]]:expression lambda 是 Python 预留的关键字，[arg…] 和 expression 由用户自定义。
例如 最简单 的写法  
ss = lambda x, y: x * y
print(ss(3,4))
相当于声明了一个方法 只不过是没有名字的 而且lambda 函数有输入和输出，拥有自己的命名空间
-------- lambda 的高阶函数用法 --------
map() 函数 ：
map() 会根据提供的函数对指定序列做映射 
第一个参数 function 以参数序列中的每一个元素调用 function 函数，返回包含每次 function 函数返回值的迭代器。
map(lambda x: x ** 2, [1, 2, 3, 4, 5])
reduce() 函数 ：
reduce() 函数会对参数序列中元素进行累积
from functools import reduce
reduce(lambda x, y: x + y, [1, 2, 3, 4, 5])
print(qq)
sorted() 函数：
sorted() 函数对所有可迭代的对象进行排序操作 
------------   例如 对 列表嵌套列表的第3个值进行排序
L = [['a', 135252,11], ['c', 12413,4], ['d', 13134,3],['b', 1313412,9]]
result = sorted(L, key=lambda x: x[2])
print(result)
### 结果----- 
[['d', 13134, 3], ['c', 12413, 4], ['b', 1313412, 9], ['a', 135252, 11]]
------------   例如 对 字典的某一个 key 的值进行排序 ----
versions = [
            {'version':'1.0','release_time':1652354611},
            {'version':'1.1','release_time':1652354618},
            {'version':'1.2','release_time':1652354718},
            {'version':'1.3','release_time':1652354918},
            {'version':'1.4','release_time':1652355618},
        ]
ss = sorted([{'version': v['version'], 'release_time': v['release_time']} for v in versions],
               key=lambda x: x['release_time'], reverse=True)

print(ss)
lambda 后面一定要是一个 可迭代的对象 这样才可以

current_ranks = {
'135451': 1,
'141351': 2,
'637612': 3,
'2747475': 4,
'903745': 5,
'652': 6,
'3515151': 7,
'42314': 8,
'5161': 9,
'22': 10,
'414515': 11,
}
last_ranks = {
'135451': 1,
'561458': 2,
'637612': 3,
'1479': 4,
'903745': 5,
'1141515': 6,
'3515151': 7,
'6541': 8,
'5161': 9,
'22': 10,
'414515': 11,
}
add_apps = list(set(current_ranks.keys()).difference(
set(last_ranks.keys())))
add_apps = sorted(add_apps, key=lambda x: list(
current_ranks.keys()).index(x))
print(add_apps)
例如 获取 上一次请求的排名和当前的排名 取差集 获得 这次在榜单上一次不在榜单app 在按照 排名排列
add_apps = sorted(add_apps, key=lambda x: current_ranks[str(x)]) 也可以这样写
```

