
---
title: "Mysql 介绍2"
date: 2023-03-23
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



### 关于mysql介绍2

#### 1: 数据库的列类型

> 数值

1. Tinyint  十分小的数据 1个字节  

2. smallint  较小的数据  2个字节
3. int            标准的整数  4个字节
4. bigint       较大的数据  8个字节
5. float         浮点数          4个字节
6. Decimal   字符串形式的浮点数  金融计算

> 字符串

1. Char     比较小的字符串    0-255
2. Varchar 可变字符串           0-65535
3. Tinytext   微型文本            2^8 -1
4. Text          文本串                2^16 -1

> 时间

1. data  YYYY-MM-DD
2. time  HH-mm-ss
3. datatime YYYY-MM-DD:HH-mm-ss
4. Timestamp 时间戳 
5. year 年份表示

> null

没有值 ，未知

注意 不要使用null进行计算 结果为 null



#### 2：数据库的字段属性

Unsigned :  无符号的整数  声明了该列不能声明为负数

zerofill :  0填充的  不足的位数 用0来填充   int(3)  ,5 --- 005

自增 : 通常理解为自增 自动在上一条的基础上 +1 (默认) 通常设计主键id 可以设置起始值

非空 ： bull 和 not null   如果设置了 not null  不给值就报错

默认： 设置默认值， 例如 sex  男  女

每一个表 都必须存在五个字段   id  主键 version 乐观锁 is_delete 伪删除 创建时间 更新时间



#### 3：mylsam 和 innodb的区别

```
---常用命令---
show create database school
show create table student
desc student

常用MySQL引擎, 分别是MyISAM、InnoDB、Memory和CSV
MySAM是MySQL 5.5.8版本之前的默认引擎, 在MySQL 5.5.8+后的默认存储引擎是InnoDB。
InnoDB 的特点 ：支持事务 默认使用行级锁 支持外键约束 5.7版本后支持全文检索与空间函数 
在InnoDB中, 只有利用索引的更新, 删除操作才会使用行级锁, 不能使用索引的写操作则是表级锁
MyISAM Memory 都不支持事务 不支持外键约束

MyISAM 支持全文索引 它的表空间是小的 在InnoDB中 不支持全文索引  它的表空间 是比较大的

常规使用操作 ：
myisam  节约空间  速度较快
innodb  安全性高 事物处理 多表多用户操作
数据库 的存储本质还是 文件的存储

需要设置数据库表的字符集编码为utf-8
```

#### 4： 修改和删除数据表字段

> 修改  旧表名到新 表名
>
> alter  table  表名 rename as 新名称
>
> 增加表的字段
>
> Alter table 表名 add age INT(11)
>
> 修改表的字段  修改约束  change 字段重命名 
>
> Alter table 表名 modify age varchar(11)
>
> modify是用来修改列属性中较微小的操作的,例如修改列的属性,重命名时无法使用modify
>
> change是用来修改列属性中幅度变化较大的操作的,例如修改列名
>
> 注意:使用change只修改列属性不修改列名时,需将列名写俩遍,因为change一使用就必须修改列名,不建议仅修改列属性时使用change
>
> 为什么修改列属性是微小的操作,而修改列名是幅度变化较大的操作呢?
>
> 注意:修改列名对于表来说牵扯的东西会更多一点,而修改列属性仅仅针对了这一列而言,因此修改列属性是较小的操作
>
> 建议需要修改列名时再使用change,修改列属性时只使用modify

-- 假设id本来没有非空约束,增加非空约束:
ALTER TABLE name modify id int not null;

-- 使用change同样修改id属性:增加非空约束
ALTER TABLE name change id id int not null;

-- 修改id列名为Sno:
ALTER TABLE name change id Sno int not null;

> 删除 表的字段
>
> Alter table 表名 drop age
>
> 如果 表存在就删除 表
>
> Drop table  if  exists  表名

#### 5：DML 语言

数据库意义：数据存储，数据管理

DML语言 ： 数据操作语言

insert update delete

> insert

Insert info 表名 （[字段1,字段2, ....]）values('值1','值2', .....)  一般来说如果主键自增 可以省略

插入多条数据  Insert info 表名 （[字段1,字段2, ....]）values('值1','值2', .....) ,values('值1','值2', .....),values('值1','值2', .....)....

只需要在value 后新增即可 字段可以省略 但是值是需要一一对应的

> update

Update  表名 set name="新名字" where Id=1

修改多个属性  

update 表名 set 'name'=‘新名字’,'address'='test@163.com' ....   where  Id=1

> delete

Delete from 表名 where 条件

>truncate 表名  完全清空一个数据表 表的结构索引不会改变

truncate 不会影响事物， 重新设置 自增的主键id 计数器会归为0





