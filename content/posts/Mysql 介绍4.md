
---
title: "Mysql 介绍4"
date: 2023-05-01
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



### Mysql 介绍4

#### 1自连接

```
上一篇文章讲过了 左右联表和inner join  本篇会讲一下 自连接

自连接的核心就是 一张表 拆成了 两个一样的表即可

自连接什么时候用？

看下面的例题
例1：查询同时选修了c01和c04的学生的学号(sc)
例2：查询选修了课程c01或c04的学生的学号

错误的写法是 
1、SELECT sno FROM sc WHERE cno="c01" AND cno="c04";
2、SELECT sno FROM sc WHERE cno="c01" OR cno="c04";

使用自连接的正确的写法：
SELECT a.sno FROM sc a,sc b WHERE a.sno=b.sno AND a.cno="c01" AND b.cno="c04";

例3：查询c01的课程比c04课程高的学生的学号以及c01和c04对应的课程成绩；

SELECT a.sno,a.cno,a.degree,b.cno,b.degree FROM sc a,sc b 
WHERE a.sno=b.sno AND a.cno="c01" AND b.cno="c04" AND a.degree>b.degree;

什么时候使用自连接：

(1)条件发生在同一列（同一个字段）；
回顾刚刚的题目：发现要求的两个条件，课程为c01和c04发生在同一列
自连接的思路相似于内连接：
(1)找到涉及的表
(2)找到具体相同意义的字段，作为关系字段
(3)加上筛选条件：连接之后的多张表看作一张新表加上筛选条件
注：自连接时需要把一张表看作两张表；并且自连接一定要给表起别名；

```

#### 2:分页和排序

```
limit 分页 排序 group by 

select [all | distinct ] {* | table.* | [table.field1 [as alias1] [,table.field2 [as alias2]] [,...]]}
from table_name [as table_alias] [left |right |inner join table_name2 ] [ where ...]
[group by ] [having ] [order by] [limit] {[offset,] row_count | row_countOFFSET offset }];

```

注意：[] 括号代表可选的，{} 括号代表必选项  顺序是有严格的要求的

```
排序就俩 升序 asc 降序 desc 

select s.name,s.result.s.type from student s where s.subjet='语文' order by s.result desc

分页 每页10条 limit 在数据查询的最后面 两个参数 起始值 页面的大小 
select s.name,s.result.s.type from student s where s.subjet='语文' order by s.result desc
limit 0，10

```

#### 3:子查询和嵌套查询

```
where ( 嵌套一个查询语句 )
本质就是 在 where 中 的值 从固定值 变成一个变动值

定义：一个内层查询语句(select-from-where)块可以嵌套在另外一个外层查询块的where子句中，其中外层查询也称为父查询，主查询。内层查询也称子查询，从查询

子查询一般不使用order by子句，只能对最终查询结果进行排序

执行过程：
1 ---  执行子查询，其结果不被显示，而是传递给外部查询，作为外部查询的条件使用
2 ---  执行外部查询，并显示整个结果

1:单行子查询：单行子查询是指子查询的返回结果只有一行数据。当主查询语句的条件语句中引用子查询结果时可用单行比较符号（＝, >, <, >=, <=, <>）来进行比较

2:多行子查询：多行子查询即是子查询的返回结果是多行数据

3:多列子查询：当是单行多列的子查询时，主查询语句的条件语句中引用子查询结果时可用单行比较符号（＝, >, <, >=, <=, <>）来进行比较；当是多行多列子查询时，主查询语句的条件语句中引用子查询结果时必须用多行比较符号（IN，ALL,ANY）来进行比较。

相关子查询:

执行过程：

解释：相关子查询无法独立于外部查询而得到解决，该子查询需要一个‘类编号’的值，而该值根据表中不同行而改变

求解相关子查询不能像求解普通子查询那样，一次将子查询求解出来，然后求解父查询。相关子查询由于必须与外层查询有关，因此必须反复求值

外层先传进去值，内部条件筛选后再传出筛选过后的值

执行过程：

从外层查询中取出一个元组，将元组相关列的值传给内层查询
执行内层查询，得到子查询操作的值
外查询根据子查询返回的结果或结果集得到满足条件的行
外查询取出下一个元组重复做步骤1-3，直到外层的元组全部处理完毕

```



#### 4 mysql 函数

```
官方文档 ：  https://dev.mysql.com/doc/

---------------------------常用函数（并不常用）---------------------------
数学运算 
select abs(-1) 绝对值     1
select ceiling(9.4)  向上取整      10
select floor(9.4)  向下取整        9
select Rand()  返回一个0-1之间的随机数
select sign()  判断一个数的符号。正或者负 负数返回-1 整数返回 1    0返回0
字符串函数
select char_length() 返回字符串的长度
select concat('我','爱',你')  拼接字符串
select lower('ssss')  全部小写字母
select upper(‘ssss')  全部大写字母


---------------------------聚合函数---------------------------

count() 计数 SUM() 求和 AVG() 平均值 MAX() 最大值 MIN() 最小值

select count(1) from ss   # 不会忽略所有的 null值  本质是按行去做的
select count(a) from ss   # 会忽略所有的 null值
select count(*) from ss   # 不会忽略所有的 null值 


```

#### 5 事务

事务简单来说就是要么都成功 要么都失败  例如转账 

事务原则 就是ACID. 原子性 一致性 隔离型 持久性 会产生 各种问题 例如 脏读 幻读等各种问题

这个问题之前的文章有介绍过 那就在重复一遍 



原子性(atomicity)： 要么都成功。要么都不成功

一致性(consistency)： 事务前后的数据完整性要保证一致

持久性(durability)：事物一旦提交不可逆，被持久化到数据库中

隔离型(isolation)：事务的隔离型是多个用户并发访问数据库时，数据库每一个用户开启的事物，不能被其他事物的操作数据所干扰

多个事务之间要相互隔离

隔离所产生的问题

脏读 ： 指一个事务读取了另外一个事务未提交的数据

不可重复读： 在一个事务内读取表中的某一行数据，多次读取的结果不同

虚读 是指在一个事务内读取到了别的事务插入的数据，导致前后读取不一致。

#### 6:索引

```
mysql 的一个特点就是索引 
mysql 官方对索引的定义为： 索引（index) 是帮助mysql高效获取数据的数据鹅结构。提取句子主干，就可以得到索引的本质：索引是数据结构
索引的分类
主键索引  parmary key   唯一的标识，主键不可重复 只能有一个列作为主键
唯一索引  unique key  避免重复的列出现，唯一索引可以重复，多个列都可以标识唯一索引
常规索引 key index    默认的   
全文索引 fulltext     在特定的引擎下才有

索引原则 ：
索引不是越多越好 不要对经常变动的数据加索引 索引一般加在常用的查询字段上。 默认索引是b树
具体可以查看此链接 ：  http://blog.codinglabs.org/articles/theory-of-mysql-index.html
```

#### 7 规范的数据设计

当数据比较复杂的时候，需要设计 这个就要需要实际的项目去实践

#### 8 三大范式

如果不数据规范化 可能会造成 信息重复，更新日常，插入异常

```
第一范式 要求数据库表的每一列都是不可分割的院子数据项，
第二范式：前提满足第一范式 每张表只描述一件事
第三范式：满足第一和第二范式 确保数据库中的每一列数据都和主键直接相关而不能间接相关。

```

其实三大范式 是为了规范数据库的设计









