---
title: "python collections "
date: 2022-06-28
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

### python 基础 5

#### 1: collections 

```
from collections import Counter  
# 计数器功能
s = "can you speak chinese"
a = Counter(s)
print(a)
# 结果 
Counter({' ': 3, 'e': 3, 'c': 2, 'a': 2, 'n': 2, 's': 2, 'y': 1, 'o': 1, 'u': 1, 'p': 1, 'k': 1, 'h': 1, 'i': 1})
print(dict(a))  直接转换成字段 # 结果 
{'c': 2, 'a': 2, 'n': 2, ' ': 3, 'y': 1, 'o': 1, 'u': 1, 's': 2, 'p': 1, 'e': 3, 'k': 1, 'h': 1, 'i': 1}
其中 s 可以是 字符串\列表\元祖\字典 等 可迭代的对象

most_common(n) (统计出现最多次数的n个元素)
print(a.most_common(4)) 代表 出现次数最多的四个元素 
elements (返回一个Counter计数后的各个元素的迭代器)
for i in a.elements():
    print(i, end="")
update (类似集合的update,进行更新)
a.update("iusepython")
print(a)  # 结果
Counter({'e': 4, 'n': 3, ' ': 3, 's': 3, 'c': 2, 'a': 2, 'y': 2, 'o': 2, 'u': 2, 'p': 2, 'h': 2, 'i': 2, 'k': 1, 't': 1})

import collections 
此模块 是 数据类型容器模块 里面有一个collections.defaultdict()经常被用到 下面就来介绍这个东西
defaultdict属于内建函数dict的一个子类，调用工厂函数提供缺失的值
里面可以 放置 各种工厂函数 比如 int(), long(), float(), complex()str(), unicode(), basestring()
list(), tuple() ,type(), dict()
----------------------------------例子1---------------------------------
import collections
s = [( 'yellow' , 1 ), ( 'blue' , 2 ), ( 'yellow' , 3 ), ( 'blue' , 4 ), ( 'red' , 1 )]
d = collections.defaultdict( list )
for k, v in s:
     d[k].append(v)
print(list (d.items()))  # 结果 
[('yellow', [1, 3]), ('blue', [2, 4]), ('red', [1])]

----------------------------------例子2---------------------------------
import collections

s = [('yellow', 1), ('blue', 2), ('yellow', 3), ('blue', 4), ('red', 1)]
# defaultdict
d = collections.defaultdict(list)

for k, v in s:
    d[k].append(v)
# Use dict and setdefault
g = {}
for k, v in s:
    g.setdefault(k, []).append(v)

# Use dict
e = {}
for k, v in s:
    e[k] = v

print(dict (d.items()))
print(dict (g.items()))
print(dict (e.items()))
结果 ： 
{'yellow': [1, 3], 'blue': [2, 4], 'red': [1]}
{'yellow': [1, 3], 'blue': [2, 4], 'red': [1]}
{'yellow': 3, 'blue': 4, 'red': 1}
可以看出来 collections.defaultdict(list)使用起来效果和运用dict.setdefault()比较相似 
这种方法会和dict.setdefault()等价，但是要更快。

```

#### 2:calendar

```
python当中的日历模块提供了对日期的一系列操作方法，并且可以生成日历，代码如下
import calendar
print(calendar.calendar(2022))
```

