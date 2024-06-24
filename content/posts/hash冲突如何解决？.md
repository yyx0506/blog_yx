
---
title: "hash冲突如何解决"
date: 2022-05-08
categories:
- tranquilpeak
- features
tags:
- hash
# keywords:
# - hash

thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->


### hash冲突如何解决？

#### hashmap 原理

hashmap底层是采用数组的结构来存储数据，数组的默认长度是16，当我们通过put方法去添加数据的时候，ha shmap 会通过key 的hash值去进行取模运算，最终把这样一个值保存到数组的一个指定位置，但是这样一个方式会存在hash冲突的问题，也就是说两个hash不同值的key最后取模以后会落到同一个数组下标，所以hashmap引入了一个链式寻址法来解决hash冲突的问题，对这些产生冲突的key,hashmap会把这些key组成一个单向链表，然后采用尾插法将这样一个key保存到链表的一个尾部，另外为了避免链表过长，导致查询效率下降，所以当链表长度大于8，数组长度大于64的时候，hashmap会把当前链表转化为红黑树，从而去减少链表数据查询的一个时间复杂度来提升查询速度。

#### hashmap冲突解决方法

解决hash冲突的方法有很多，第一个再hash法，比如布隆过滤器，第二个开放寻址法：直接从冲突的数据位置往下寻找一个空的数据组下表进行数据存储。3:建立公共溢出去，也就是把存在冲突的key统一放在一个公共溢出区里面