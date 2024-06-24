
---
title: "Mysql 介绍3"
date: 2023-03-24
categories:
- tranquilpeak
- features
tags:
- mysql
# keywords:
# - mysql

thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->

### myslq介绍3

### 1:查询 DQL 查询数据

```
-------------------------指定查询字段-------------------------
select * from 表名
select name from 表名
-- 别名 给字段或者表 起一个名字 as
select name as 名字 from 表名 as sss
-- 函数 concat (a,b) 拼接符
select concat('姓名 ：',name) as 新名字 from 表名

-- 去重 distinct 
select distinct 'name' from 表名  对字段重复数据进行去重

数据库的列（表达式）
select Version() -- 查询系统版本（函数）
select 'name','result'+1 as '+1后的结果' from SSS

select 表达式 from 表名

-------------------------where 条件字句-------------------------
作用： 检索数据中 符合条件 的值 
运算符 与或非  and &&  or ||  not ! 尽量使用英文字母

select * from app_infos where app_id>10000 and app_id <11000

select * from app_infos where app_id between 10000 and 11000

模糊查询 比较运算符

is null  is not null 为空 或者不为空  返回的都是不二之选 
between and  前后都是闭区间
like  a like b  如果 a匹配到b 则结果为真
in  a in (a1,a2,a3,a4) 在某个区间

like 可以 结合 %（代表0到任意个字符）_(一个字符)

select name from sss where name like '王__'


in 是一个具体的一个或者多个值 不能和% 一起


-------------------------联表查询-------------------------
JOIN 对比
LEFT JOIN RIGHT JOIN  INNERT JOIN  如下图所示
```


![](/img/mysql1.png)


```
确定交叉点 
inner join 如果表中有一个匹配 就返回行
left join 会从左表中返回所有值 即使右表没有匹配
right join 会从右表中返回所有值 即使左表没有匹配

--- join on 连接查询。或者 where 等值查询
select a.app_id,a.app_name,a.app_desc from app_info as a left join app_download  as d where a.app_id = d.app_id

select a.app_id,a.app_name,a.app_desc from app_info as a left join app_download  as d on a.app_id = d.app_id

分析需求 分析查询的字段来自那些表 确定交叉点  + 判断条件

select studentNo,studentName,subjectName,studentresult from student s right join result as r on 
r.studentno = s.studentno  inner join subject as sub on r.subjectNo = sub.subjectno
```



### 2:Mysql 性能优化

##### 1:数据结构优化

良好的逻辑设计和物理设计是高性能的基石

###### 1.1数据类型优化

数据类型优化的基本原则

```
---- 更好的通常更好 - 越小的数据类型通常会更快，占用更少的磁盘，内存，处理时需要的cpu周期也更少。

例如: 整型比字符类型操作代价低，因而会使用整型来存储ip地址，使用datetime来存储时间，而不是使用字符串。

----简单就好 - 如整型比字符型操作代价低

例如 ： 很多软件会用整型来存储ip地址。
例如 ： unsigned 表示不允许负值，大致可以使正数的上限提高一倍。

---- 尽量避免null - 可为null的列会使得索引，索引统计和值比较都更复杂。
```

类型的选择

```
整数类型通常是标识列最好的选择，因为他们很快并且可以使用 AUTO_INCREMENT
ENUM 和 SET 类型通常是一个糟糕的选择，应尽量避免
应该尽量避免用字符串作为标识列，因为它们很消耗空间，并且通常比数字类型慢。对于 MD5，SHA，UUID这类随机字符串，由于比较随机，所以可能
分布在很大的空间内，导致 INSERT 及一些 SELECT 语句变得很慢。
如果存储UUID,应该移除 - 符号；更好的做法是，用 UNHEX 函数转换UUID 为16字节的数字，并存储在一个 BINARY 的列中，
检索时 可以通过 HEX 函数来格式化为16进制格式。
```

###### 1.2表设计

```
太多的列 - 设计者为了图方便，将大量冗余列加入表中，实际查询中，表中很多列是用不到的。这种宽表模式设计，会造成不小的性能代价，尤其是 ALTER TABLE 非常耗时。
太多的关联 - 所谓的实体 - 属性 - 值（EVA）设计模式是一个常见的糟糕设计模式。Mysql 限制了每个关联操作最多只能有 61 张表，但 EVA 模式需要许多自关联。
枚举 - 尽量不要用枚举，因为添加和删除字符串（枚举选项）必须使用 ALTER TABLE。
尽量避免 NULL
```

###### 1.3 范式和反范式

**范式化目标是尽量减少冗余，而反范式化则相反**。

范式化的优点：

- 比反范式更节省空间
- 更新操作比反范式快
- 更少需要 `DISTINCT` 或 `GROUP BY` 语句

范式化的缺点：

- 通常需要关联查询。而关联查询代价较高，如果是分表的关联查询，代价更是高昂。

在真实世界中，很少会极端地使用范式化或反范式化。实际上，应该权衡范式和反范式的利弊，混合使用。

###### 1.4索引优化

#### 何时使用索引

- 对于非常小的表，大部分情况下简单的全表扫描更高效。
- 对于中、大型表，索引非常有效。
- 对于特大型表，建立和使用索引的代价将随之增长。可以考虑使用分区技术。
- 如果表的数量特别多，可以建立一个元数据信息表，用来查询需要用到的某些特性。

#### 索引优化策略

- 索引基本原则
  - 索引不是越多越好，不要为所有列都创建索引。
  - 要尽量避免冗余和重复索引。
  - 要考虑删除未使用的索引。
  - 尽量的扩展索引，不要新建索引。
  - 频繁作为 `WHERE` 过滤条件的列应该考虑添加索引。
- **独立的列** - “独立的列” 是指索引列不能是表达式的一部分，也不能是函数的参数。
- **前缀索引** - 索引很长的字符列，可以索引开始的部分字符，这样可以大大节约索引空间。
- **最左匹配原则** - 将选择性高的列或基数大的列优先排在多列索引最前列。
- **使用索引来排序** - 索引最好既满足排序，又用于查找行。这样，就可以使用索引来对结果排序。
- `=`、`IN` 可以乱序 - 不需要考虑 `=`、`IN` 等的顺序
- **覆盖索引**
- **自增字段作主键**

## # https://blog.csdn.net/qq_34967770/article/details/127774607 关于sql优化

