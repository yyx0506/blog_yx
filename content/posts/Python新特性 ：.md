---
title: "Python新特性"
date: 2022-05-06
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


### Python新特性 ：

#### 1: 字典新特性

```
x = {"key1": "value1 from x", "key2": "value2 from x"}
y = {"key2": "value2 from y", "key3": "value3 from y"}
print(x | y)
{'key1': 'value1 from x', 'key2': 'value2 from y', 'key3': 'value3 from y'}
print(y | x)
{'key2': 'value2 from x', 'key3': 'value3 from y', 'key1': 'value1 from x'}

```

#### 2:一些关键字

```
any() #意思是：判断一个tuple或者list是否全部为空、0、False。如果全为空、0、False，则返回False；如果（只要有非[空或0或False]）不全为空、0、False，则返回True。 
注意：空tuple（小括号）和空list（中括号）、空字典dictionary空集合set（大括号）的返回值是False。
例如 : a = [0,0,'',False] any(a) == False
zip() # 函数⽤于将可迭代的对象作为参数，将对象中对应的元素打包成⼀个个元组，然后返回由这些元组组成的对象，这样做的好处是节约了不少的内存,可以通过list或者dict套zip 组成一个新的可迭代参数
例如 a = [1,2,3] b= [4,5,6]
list(zip(a,b)) = [(1, 4), (2, 5), (3, 6)] dict(zip(a,b)) = {1: 4, 2: 5, 3: 6}

```

