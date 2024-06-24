
---
title: "go 的ORM"
date: 2023-07-13
categories:
- tranquilpeak
- features
tags:
- golang
# keywords:
# - golang
thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->

#### go 的ORM

#### 1:XORM和GORM的区别

Xorm和GORM是两个流行的Go语言对象关系映射（ORM）库，用于简化数据库访问和操作。

Xorm是一个轻量级的ORM库，支持多种关系型数据库，包括MySQL、PostgreSQL、SQLite、Microsoft SQL Server等。它提供了丰富的特性，如自动映射数据库表和Go结构体、事务支持、查询构建器、缓存等。Xorm具有较高的灵活性和可定制性，可以根据需求进行调整和扩展。

GORM（全称为Go ORM）是另一个流行的Go语言ORM库，目标是提供简单、强大且易于使用的数据库操作接口。它支持多种数据库，包括MySQL、PostgreSQL、SQLite和Microsoft SQL Server等。GORM提供了类似于Active Record的查询语法和链式调用，使得编写和执行数据库查询变得简单和直观。它还提供了一些额外的功能，如数据库迁移、事务支持、预加载关联数据等。

虽然Xorm和GORM都是用于Go语言的ORM库，它们在设计和使用方式上略有不同。Xorm更加注重灵活性和可定制性，适用于复杂的数据库操作场景，而GORM则更加注重简洁性和易用性，适用于快速开发和简单的查询操作。选择使用哪个库取决于个人喜好和项目需求，你可以根据自己的情况选择适合的库进行数据库操作。

#### 2:关于gorm

官方文档链接https://gorm.io/zh_CN/docs/index.html

作者是一位国人，非常强大

特性

- 全功能 ORM
- 关联 (Has One，Has Many，Belongs To，Many To Many，多态，单表继承)
- Create，Save，Update，Delete，Find 中钩子方法
- 支持 `Preload`、`Joins` 的预加载
- 事务，嵌套事务，Save Point，Rollback To Saved Point
- Context、预编译模式、DryRun 模式
- 批量插入，FindInBatches，Find/Create with Map，使用 SQL 表达式、Context Valuer 进行 CRUD
- SQL 构建器，Upsert，数据库锁，Optimizer/Index/Comment Hint，命名参数，子查询
- 复合主键，索引，约束
- Auto Migration
- 自定义 Logger
- 灵活的可扩展插件 API：Database Resolver（多数据库，读写分离）、Prometheus…
- 每个特性都经过了测试的重重考验
- 开发者友好

第一步安装

```
go get -u gorm.io/gorm
go get -u gorm.io/driver/sqlite
```

#### 3:gorm的应用

快速入门

```go
//首先我们先写一个gorm 单例连接代码

package gormMysql

import (
	"fmt"
	"gorm.io/driver/mysql"
	"gorm.io/gorm"
	"sync"
)

var (
	dbInstances map[string]*gorm.DB
	once        sync.Once
	lock        sync.Mutex
)

// DatabaseConfig 存储数据库连接配置信息
type DatabaseConfig struct {
	Domain   string
	Port     string
	Username string
	Password string
	Database string
}

func GetDBInstance(config DatabaseConfig) (*gorm.DB, error) {
	var err error

	once.Do(func() {
		dbInstances = make(map[string]*gorm.DB)
	})

	lock.Lock()
	defer lock.Unlock()

	key := fmt.Sprintf("%s:%s@tcp(%s:%s)/%s", config.Username, config.Password, config.Domain, config.Port, config.Database)
	if db, ok := dbInstances[key]; ok {
		return db, nil
	}

	// 创建数据库连接
	dsn := fmt.Sprintf("%s:%s@tcp(%s:%s)/%s?charset=utf8mb4&parseTime=True&loc=Local",
		config.Username, config.Password, config.Domain, config.Port, config.Database)
	dbInstance, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
	if err != nil {
		fmt.Println("创建数据库连接失败:", err)
		return nil, err
	}

	dbInstances[key] = dbInstance

	return dbInstance, nil
}

var (
	GormMysqlConfigs = map[string]DatabaseConfig{
		"config1": {
			Domain:   "example1.com",
			Port:     "3306",
			Username: "user1",
			Password: "password1",
			Database: "database1",
		},
    // 后面可以根据情况追加各种配置
	}
)

// 然后我们声明一下模型 关于模型的其他配置可以查看官网https://gorm.io/zh_CN/docs/models.html
package models

import "time"

type Users struct {
	ID          int
	Name        string
	Age         int
	Birthday    time.Time
	CreatedAt   time.Time // 在创建时，如果该字段值为零值，则使用当前时间填充
	UpdatedAt   int       // 在创建时该字段值为零值或者在更新时，使用当前时间戳秒数填充
	UpdatedNano int64     `gorm:"autoUpdateTime:nano"` // 使用时间戳纳秒数填充更新时间
	Created     int64     `gorm:"autoCreateTime"`      // 使用时间戳秒数填充创建时间

}
// 关于字段的控制 gorm 也做了要求
type Users struct {
  Name string `gorm:"<-:create"` // 允许读和创建
  Name string `gorm:"<-:update"` // 允许读和更新
  Name string `gorm:"<-"`        // 允许读和写（创建和更新）
  Name string `gorm:"<-:false"`  // 允许读，禁止写
  Name string `gorm:"->"`        // 只读（除非有自定义配置，否则禁止写）
  Name string `gorm:"->;<-:create"` // 允许读和写
  Name string `gorm:"->:false;<-:create"` // 仅创建（禁止从 db 读）
  Name string `gorm:"-"`  // 通过 struct 读写会忽略该字段
  Name string `gorm:"-:all"`        // 通过 struct 读写、迁移会忽略该字段
  Name string `gorm:"-:migration"`  // 通过 struct 迁移会忽略该字段
}

// 下面我们创建一个表并使用一下
package main

import (
	"fmt"
	"go_preject/internal/gormMysql"
	"go_preject/models"
	"time"
)

func main() {
	db, err := gormMysql.GetDBInstance(gormMysql.GormMysqlConfigs["user_yx"])
	if err != nil {
		panic(err)
	}
	db.AutoMigrate(&models.Users{})
	user := models.Users{Name: "Jinzhu", Age: 18, Birthday: time.Now()}

	result := db.Create(&user) // 通过数据的指针来创建

	fmt.Println(user.ID)             // 返回插入数据的主键
	fmt.Println(result.Error)        // 返回 error
	fmt.Println(result.RowsAffected) // 返回插入记录的条数
	
}
// 官方文档的一些操作 GORM 提供了 First、Take、Last 方法，以便从数据库中检索单个对象。当查询数据库时它添加了 LIMIT 1 条件，且没有找到记录时，它会返回 ErrRecordNotFound 错误
	var user1 models.Users
	db.First(&user1, 1) // 根据整型主键查找
	fmt.Println(user1)
	var user2 models.Users
	db.First(&user2, "name = ?", "Jinzhu") // 查找 code 字段值为 D42 的记录
	fmt.Println(user2)

	// Update - 将 user2 的 age 更新为 200
	db.Model(&user2).Update("age", 200)
	fmt.Println(user2)
	// Update - 更新多个字段
	db.Model(&user2).Updates(models.Users{Age: 150, Name: "F42"})
	fmt.Println(user2)
	db.Model(&user1).Updates(map[string]interface{}{"age": 100, "name": "F43"})
	fmt.Println(user2)
	// Delete - 删除 product
	db.Delete(&user2, 1)

//根据主键检索 
db.First(&user, 10)
// SELECT * FROM users WHERE id = 10;

db.First(&user, "10") // 例如传入的是 >=10 就会造成sql注入
// SELECT * FROM users WHERE id = 10;
// 为了防止错误的信息进来 需要加入判断
userInputID := "1=1;drop table users;"
// 安全的，返回 err
id,err := strconv.Atoi(userInputID)
if err != nil {
    return error
}
db.First(&user, id)

db.Find(&users, []int{1,2,3})
// SELECT * FROM users WHERE id IN (1,2,3);
// 那么为了防止sql 注入 gorm 都做了哪些操作呢
//用户的输入只能作为参数，例如：

// 如果你的主键就是key的话 建议按照下面的写法来写
db.First(&user, "id = ?", "1b74413f-f3b8-409f-ac47-e8c062e3472a")

userInput := "jinzhu;drop table users;"

// 安全的，会被转义
db.Where("name = ?", userInput).First(&user)

// SQL 注入
db.Where(fmt.Sprintf("name = %v", userInput)).First(&user)
// 为了支持某些功能，一些输入不会被转义，调用方法时要小心用户输入的参数。 例如

db.Select("name; drop table users;").First(&user)
db.Distinct("name; drop table users;").First(&user)

db.Model(&user).Pluck("name; drop table users;", &names)

db.Group("name; drop table users;").First(&user)

db.Group("name").Having("1 = 1;drop table users;").First(&user)

db.Raw("select name from users; drop table users;").First(&user)

db.Exec("select name from users; drop table users;")

db.Order("name; drop table users;").First(&user)

// 查询所有数据是 声明一个切片即可
	users := []models.Users{}
	db.Find(&users)
	fmt.Println(users)

// -------------按照条件检索的-------------

// Get first matched record
db.Where("name = ?", "jinzhu").First(&user)
// SELECT * FROM users WHERE name = 'jinzhu' ORDER BY id LIMIT 1;

// Get all matched records
db.Where("name <> ?", "jinzhu").Find(&users)
// SELECT * FROM users WHERE name <> 'jinzhu';

// IN
db.Where("name IN ?", []string{"jinzhu", "jinzhu 2"}).Find(&users)
// SELECT * FROM users WHERE name IN ('jinzhu','jinzhu 2');

// LIKE
db.Where("name LIKE ?", "%jin%").Find(&users)
// SELECT * FROM users WHERE name LIKE '%jin%';

// AND
db.Where("name = ? AND age >= ?", "jinzhu", "22").Find(&users)
// SELECT * FROM users WHERE name = 'jinzhu' AND age >= 22;

// Time
db.Where("updated_at > ?", lastWeek).Find(&users)
// SELECT * FROM users WHERE updated_at > '2000-01-01 00:00:00';

// BETWEEN
db.Where("created_at BETWEEN ? AND ?", lastWeek, today).Find(&users)
// SELECT * FROM users WHERE created_at BETWEEN '2000-01-01 00:00:00' AND '2000-01-08 00:00:00';

// -------------还可以 通过 Struct 和map 来进行检索-------------
db.Where(&User{Name: "jinzhu", Age: 20}).First(&user)
// SELECT * FROM users WHERE name = "jinzhu" AND age = 20 ORDER BY id LIMIT 1;

// Map
db.Where(map[string]interface{}{"name": "jinzhu", "age": 20}).Find(&users)
// SELECT * FROM users WHERE name = "jinzhu" AND age = 20;

// Slice of primary keys
db.Where([]int64{20, 21, 22}).Find(&users)
// SELECT * FROM users WHERE id IN (20, 21, 22);

// 当使用结构体去检索时gorm 只会查询非零值字段。这意味着 所有的“” 0 False 都不会被用于查询条件 例如
db.Where(&User{Name: "jinzhu", Age: 0}).Find(&users)
// SELECT * FROM users WHERE name = "jinzhu";
//要想使用 包含0值的条件时 可以使用map 
db.Where(map[string]interface{}{"Name": "jinzhu", "Age": 0}).Find(&users)
// SELECT * FROM users WHERE name = "jinzhu" AND age = 0;

// -------------指定结构体查询字段-------------
db.Where(&User{Name: "jinzhu"}, "name", "Age").Find(&users)
// SELECT * FROM users WHERE name = "jinzhu" AND age = 0;

db.Where(&User{Name: "jinzhu"}, "Age").Find(&users)
// SELECT * FROM users WHERE age = 0;

// -------------内联条件-------------

// Get by primary key if it were a non-integer type
db.First(&user, "id = ?", "string_primary_key")
// SELECT * FROM users WHERE id = 'string_primary_key';

// Plain SQL
db.Find(&user, "name = ?", "jinzhu")
// SELECT * FROM users WHERE name = "jinzhu";

db.Find(&users, "name <> ? AND age > ?", "jinzhu", 20)
// SELECT * FROM users WHERE name <> "jinzhu" AND age > 20;

// Struct
db.Find(&users, User{Age: 20})
// SELECT * FROM users WHERE age = 20;

// Map
db.Find(&users, map[string]interface{}{"age": 20})
// SELECT * FROM users WHERE age = 20;
// -------------Not 条件-------------
db.Not("name = ?", "jinzhu").First(&user)
// SELECT * FROM users WHERE NOT name = "jinzhu" ORDER BY id LIMIT 1;

// Not In
db.Not(map[string]interface{}{"name": []string{"jinzhu", "jinzhu 2"}}).Find(&users)
// SELECT * FROM users WHERE name NOT IN ("jinzhu", "jinzhu 2");

// Struct
db.Not(User{Name: "jinzhu", Age: 18}).First(&user)
// SELECT * FROM users WHERE name <> "jinzhu" AND age <> 18 ORDER BY id LIMIT 1;

// Not In slice of primary keys
db.Not([]int64{1,2,3}).First(&user)
// SELECT * FROM users WHERE id NOT IN (1,2,3) ORDER BY id LIMIT 1;
// -------------Or 条件-------------
db.Where("role = ?", "admin").Or("role = ?", "super_admin").Find(&users)
// SELECT * FROM users WHERE role = 'admin' OR role = 'super_admin';

// Struct
db.Where("name = 'jinzhu'").Or(User{Name: "jinzhu 2", Age: 18}).Find(&users)
// SELECT * FROM users WHERE name = 'jinzhu' OR (name = 'jinzhu 2' AND age = 18);

// Map
db.Where("name = 'jinzhu'").Or(map[string]interface{}{"name": "jinzhu 2", "age": 18}).Find(&users)
// SELECT * FROM users WHERE name = 'jinzhu' OR (name = 'jinzhu 2' AND age = 18);
// -------------选择特定字段-------------
db.Select("name", "age").Find(&users)
// SELECT name, age FROM users;

db.Select([]string{"name", "age"}).Find(&users)
// SELECT name, age FROM users;

db.Table("users").Select("COALESCE(age,?)", 42).Rows()
// SELECT COALESCE(age,'42') FROM users;

// -------------排序-------------
db.Order("age desc, name").Find(&users)
// SELECT * FROM users ORDER BY age desc, name;

// Multiple orders
db.Order("age desc").Order("name").Find(&users)
// SELECT * FROM users ORDER BY age desc, name;

db.Clauses(clause.OrderBy{
  Expression: clause.Expr{SQL: "FIELD(id,?)", Vars: []interface{}{[]int{1, 2, 3}}, WithoutParentheses: true},
}).Find(&User{})
// SELECT * FROM users ORDER BY FIELD(id,1,2,3)

// -------------Limit & Offset-------------
db.Limit(3).Find(&users)
// SELECT * FROM users LIMIT 3;

// Cancel limit condition with -1
db.Limit(10).Find(&users1).Limit(-1).Find(&users2)
// SELECT * FROM users LIMIT 10; (users1)
// SELECT * FROM users; (users2)

db.Offset(3).Find(&users)
// SELECT * FROM users OFFSET 3;

db.Limit(10).Offset(5).Find(&users)
// SELECT * FROM users OFFSET 5 LIMIT 10;

// Cancel offset condition with -1
db.Offset(10).Find(&users1).Offset(-1).Find(&users2)
// SELECT * FROM users OFFSET 10; (users1)
// SELECT * FROM users; (users2)

// -------------Group By & Having-------------
type result struct {
  Date  time.Time
  Total int
}

db.Model(&User{}).Select("name, sum(age) as total").Where("name LIKE ?", "group%").Group("name").First(&result)
// SELECT name, sum(age) as total FROM `users` WHERE name LIKE "group%" GROUP BY `name` LIMIT 1


db.Model(&User{}).Select("name, sum(age) as total").Group("name").Having("name = ?", "group").Find(&result)
// SELECT name, sum(age) as total FROM `users` GROUP BY `name` HAVING name = "group"

rows, err := db.Table("orders").Select("date(created_at) as date, sum(amount) as total").Group("date(created_at)").Rows()
defer rows.Close()
for rows.Next() {
  ...
}

rows, err := db.Table("orders").Select("date(created_at) as date, sum(amount) as total").Group("date(created_at)").Having("sum(amount) > ?", 100).Rows()
defer rows.Close()
for rows.Next() {
  ...
}

type Result struct {
  Date  time.Time
  Total int64
}
db.Table("orders").Select("date(created_at) as date, sum(amount) as total").Group("date(created_at)").Having("sum(amount) > ?", 100).Scan(&results)
// 等等其他玩法可以查看官方文档操作

// ------------------更新------------------
//Save 会保存所有的字段，即使字段是零值 
db.First(&user)

user.Name = "jinzhu 2"
user.Age = 100
db.Save(&user)
// UPDATE users SET name='jinzhu 2', age=100, birthday='2016-01-01', updated_at = '2013-11-17 21:34:10' WHERE id=111;
// 保存是一个组合功能。如果保存值不包含主键，它将执行Create，否则将执行Update（带所有字段）。例如下面执行后就是二选一
db.Save(&User{Name: "jinzhu", Age: 100})
// INSERT INTO `users` (`name`,`age`,`birthday`,`update_at`) VALUES ("jinzhu",100,"0000-00-00 00:00:00","0000-00-00 00:00:00")

db.Save(&User{ID: 1, Name: "jinzhu", Age: 100})
// UPDATE `users` SET `name`="jinzhu",`age`=100,`birthday`="0000-00-00 00:00:00",`update_at`="0000-00-00 00:00:00" WHERE `id` = 1

// ------------------更新单个列------------------
// Update with conditions
db.Model(&User{}).Where("active = ?", true).Update("name", "hello")
// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE active=true;

// User's ID is `111`:
db.Model(&user).Update("name", "hello")
// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111;

// Update with conditions and model value
db.Model(&user).Where("active = ?", true).Update("name", "hello")
// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111 AND active=true;

//------------------更新多列------------------ 也是一样struct不会更新非零值
// Update attributes with `struct`, will only update non-zero fields
db.Model(&user).Where("id = ?", 2).Updates(User{Name: "hello", Age: 18, Active: false})
// UPDATE users SET name='hello', age=18, updated_at = '2013-11-17 21:34:10' WHERE id = 111;

// Update attributes with `map`
db.Model(&user).Where("id = ?", 2).Updates(map[string]interface{}{"name": "hello", "age": 18, "active": false})
// UPDATE users SET name='hello', age=18, active=false, updated_at='2013-11-17 21:34:10' WHERE id=111;


// 关于删除
db.Delete(&User{}, 10)
// DELETE FROM users WHERE id = 10;

db.Delete(&User{}, "10")
// DELETE FROM users WHERE id = 10;

db.Delete(&users, []int{1,2,3})
// DELETE FROM users WHERE id IN (1,2,3);

// 具体可以查看文档
```

Gorm 模型和默认值

嵌入结构体

对于匿名字段，GORM 会将其字段包含在父结构体中，例如：

```
type User struct {
  gorm.Model
  Name string
}
// 等效于
type User struct {
  ID        uint           `gorm:"primaryKey"`
  CreatedAt time.Time
  UpdatedAt time.Time
  DeletedAt gorm.DeletedAt `gorm:"index"`
  Name string
}
```

对于正常的结构体字段，你也可以通过标签 `embedded` 将其嵌入，例如：

```
type Author struct {
    Name  string
    Email string
}

type Blog struct {
  ID      int
  Author  Author `gorm:"embedded"`
  Upvotes int32
}
// 等效于
type Blog struct {
  ID    int64
  Name  string
  Email string
  Upvotes  int32
}
```

并且，您可以使用标签 `embeddedPrefix` 来为 db 中的字段名添加前缀，例如：

```
type Blog struct {
  ID      int
  Author  Author `gorm:"embedded;embeddedPrefix:author_"`
  Upvotes int32
}
// 等效于
type Blog struct {
  ID          int64
  AuthorName string
  AuthorEmail string
  Upvotes     int32
}
```

### 字段标签

声明 model 时，tag 是可选的，GORM 支持以下 tag： tag 名大小写不敏感，但建议使用 `camelCase` 风格

| 标签名                 | 说明                                                         |
| :--------------------- | :----------------------------------------------------------- |
| column                 | 指定 db 列名                                                 |
| type                   | 列数据类型，推荐使用兼容性好的通用类型，例如：所有数据库都支持 bool、int、uint、float、string、time、bytes 并且可以和其他标签一起使用，例如：`not null`、`size`, `autoIncrement`… 像 `varbinary(8)` 这样指定数据库数据类型也是支持的。在使用指定数据库数据类型时，它需要是完整的数据库数据类型，如：`MEDIUMINT UNSIGNED not NULL AUTO_INCREMENT` |
| serializer             | 指定将数据序列化或反序列化到数据库中的序列化器, 例如: `serializer:json/gob/unixtime` |
| size                   | 定义列数据类型的大小或长度，例如 `size: 256`                 |
| primaryKey             | 将列定义为主键                                               |
| unique                 | 将列定义为唯一键                                             |
| default                | 定义列的默认值                                               |
| precision              | 指定列的精度                                                 |
| scale                  | 指定列大小                                                   |
| not null               | 指定列为 NOT NULL                                            |
| autoIncrement          | 指定列为自动增长                                             |
| autoIncrementIncrement | 自动步长，控制连续记录之间的间隔                             |
| embedded               | 嵌套字段                                                     |
| embeddedPrefix         | 嵌入字段的列名前缀                                           |
| autoCreateTime         | 创建时追踪当前时间，对于 `int` 字段，它会追踪时间戳秒数，您可以使用 `nano`/`milli` 来追踪纳秒、毫秒时间戳，例如：`autoCreateTime:nano` |
| autoUpdateTime         | 创建/更新时追踪当前时间，对于 `int` 字段，它会追踪时间戳秒数，您可以使用 `nano`/`milli` 来追踪纳秒、毫秒时间戳，例如：`autoUpdateTime:milli` |
| index                  | 根据参数创建索引，多个字段使用相同的名称则创建复合索引，查看 [索引](https://gorm.io/zh_CN/docs/indexes.html) 获取详情 |
| uniqueIndex            | 与 `index` 相同，但创建的是唯一索引                          |
| check                  | 创建检查约束，例如 `check:age > 13`，查看 [约束](https://gorm.io/zh_CN/docs/constraints.html) 获取详情 |
| <-                     | 设置字段写入的权限， `<-:create` 只创建、`<-:update` 只更新、`<-:false` 无写入权限、`<-` 创建和更新权限 |
| ->                     | 设置字段读的权限，`->:false` 无读权限                        |
| -                      | 忽略该字段，`-` 表示无读写，`-:migration` 表示无迁移权限，`-:all` 表示无读写迁移权限 |
| comment                | 迁移时为字段添加注释                                         |

默认值

```
type User struct {  ID   int64  Name string `gorm:"default:galeone"`  Age  int64  `gorm:"default:18"`}
```



增删改查

```go
// 创建记录并为指定的字段分配值：
db.Select("Name", "Age", "CreatedAt").Create(&user)
// INSERT INTO `users` (`name`,`age`,`created_at`) VALUES ("jinzhu", 18, "2020-07-04 11:05:21.775")
//创建记录并忽略要省略的传递字段的值： 二者正好相反
db.Omit("Name", "Age", "CreatedAt").Create(&user)
// INSERT INTO `users` (`birthday`,`updated_at`) VALUES ("2020-01-01 00:00:00.000", "2020-07-04 11:05:21.775")
// 批量插入
	users := []*models.Users{
		{Name: "Jinzhu", Age: 18, Birthday: time.Now()},
		{Name: "Jackson", Age: 19, Birthday: time.Now()},
	}

	db.Create(&users) // pass a slice to insert multiple row
	for _, user := range users {
		fmt.Println(user.ID)
	}

// 如果有大批量的数据的话。可以指定一次插入多少量的数据
var users = []User{{Name: "jinzhu_1"}, ...., {Name: "jinzhu_10000"}}
// batch size 100
db.CreateInBatches(users, 100)



```

### 



#### 4:关于xorm

官方文档链接https://xorm.io/

作者也是一位国人，非常强大

第一步安装

```
go get xorm.io/xorm
```

特性：

- 支持 Struct 和数据库表之间的灵活映射，并支持自动同步
- 事务支持
- 同时支持原始SQL语句和ORM操作的混合执行
- 使用连写来简化调用
- 支持使用ID, In, Where, Limit, Join, Having, Table, SQL, Cols等函数和结构体等方式作为条件
- 支持级联加载Struct
- Schema支持（仅Postgres）
- 支持缓存
- 通过 [xorm.io/reverse](https://xorm.io/reverse) 支持根据数据库自动生成 xorm 结构体
- 支持记录版本（即乐观锁）
- 通过 [xorm.io/builder](https://xorm.io/builder) 内置 SQL Builder 支持
- 上下文缓存支持
- 支持日志上下文

第二步查看操作文档直接开始（https://gitea.com/xorm/xorm/src/branch/master/README_CN.md）

```go
// 先找个地方 放 models 
package models

import "time"

type User struct {
	Id      int64
	Name    string
	Salt    string
	Age     int
	Passwd  string    `xorm:"varchar(200)"`
	Created time.Time `xorm:"created"`
	Updated time.Time `xorm:"updated"`
}

// 一定要导入mysql 的引擎

package main

import (
	"fmt"
	_ "github.com/go-sql-driver/mysql"
	"go_preject/models"
	"xorm.io/xorm"
)
// 设置自己的连接库
var (
	username  string = "XXXX"
	password  string = "xxxxx"
	ipAddress string = "xxxxxxx"
	port      int    = 3306
	dbName    string = "xxxxxxx"
	charset   string = "utf8mb4"
)

func main() {
	dataSourceName := fmt.Sprintf("%s:%s@tcp(%s:%d)/%s?charset=%s", username, password, ipAddress, port, dbName, charset)
	// 必须导入mysql 的驱动
	engine, err := xorm.NewEngine("mysql", dataSourceName)
	fmt.Println(engine, err)
	err = engine.Sync(new(models.User))
	fmt.Println(err)
}
// 这样我们的一个表就完成了
```

下一步老生常谈增删改 ,查单独说明

```go
// 增加一条或者多条数据

func main() {
	dataSourceName := fmt.Sprintf("%s:%s@tcp(%s:%d)/%s?charset=%s", username, password, ipAddress, port, dbName, charset)
	// 必须导入mysql 的驱动
	engine, err := xorm.NewEngine("mysql", dataSourceName)
	fmt.Println(engine, err)
	err = engine.Sync(new(models.User))
	fmt.Println(err)
	// 声明一条数据
	user := models.User{
		Id:     1,
		Name:   "sz",
		Salt:   "ssss",
		Age:    21,
		Passwd: "sssssssaqqq",
	}
  // 新增一条数据
	affected, err := engine.Insert(&user)
	fmt.Println(affected)
  // 声明一个空的列表 Insert 里面是一个空接口类型 他接受所有的参数类型
	var users []models.User
	user1 := models.User{
		Id:     3,
		Name:   "sz",
		Salt:   "ssss",
		Age:    66,
		Passwd: "sssssssaqqq",
	}
	user2 := models.User{
		Id:     4,
		Name:   "sz",
		Salt:   "ssss",
		Age:    77,
		Passwd: "sssssssaqqq",
	}
	user3 := models.User{
		Id:     5,
		Name:   "sz",
		Salt:   "ssss",
		Age:    88,
		Passwd: "sssssssaqqq",
	}
	users = append(users, user1)
	users = append(users, user2)
	users = append(users, user3)
	affectedd, errs := engine.Insert(&users)
	fmt.Println(affectedd, errs)
  // 或者 还可以指定表去 新增 id 会自增
  affected, err := engine.Table("user").Insert([]map[string]interface{}{
		{
			"name": "lunny",
			"age":  18,
		},
		{
			"name": "lunny2",
			"age":  19,
		},
	})
	fmt.Println(affected, err)
  
}

//。关于更新数据 的方法介绍

// 更新数据，除非使用Cols,AllCols函数指明，默认只更新非空和非0的字段
	user := models.User{
		Age: 124,
	}
	affected, err := engine.ID(1).Update(&user)
	fmt.Println(affected)
// 等价于 UPDATE user SET ... Where id = ?

	user := models.User{
		Age: 26,
	}
	affected, err := engine.Update(&user, &models.User{Name: "lunny"})
	fmt.Println(affected)
// 等价于 UPDATE user SET ... Where name = ?

	var ids = []int64{1, 3, 4}
	user := models.User{
		Age: 26,
	}
	affected, err := engine.In("id", ids).Update(&user)
	fmt.Println(affected)
// 等价于 UPDATE user SET ... Where id IN (?, ?, ?)

	affected, err := engine.ID(1).Cols("age").Update(&models.User{Name: "sf", Age: 12})
	fmt.Println(affected)
//等价于  UPDATE user SET age = ?, updated=? Where id = ?  注意的是即使后面的结构体重新改了名字也不会被更改只会更改cols
//里面 受影响的字段

	affected, err := engine.ID(1).Omit("name").Update(&models.User{Name: "56", Age: 36})
	fmt.Println(affected)
//等价于   UPDATE user SET age = ?, updated=? Where id = ? 修改出了name 以外的其他字段

	user := models.User{
		Salt: "qqqq",
	}
	affected, err := engine.ID(1).AllCols().Update(&user)
	fmt.Println(affected)
//等价于  UPDATE user SET name=?,age=?,salt=?,passwd=?,updated=? Where id = ?
// 需要注意的是 如果 给的一些字段没有的话 就会设置成默认值 为空或者为0


//  -------------------删除数据 --------------------------
	user := models.User{}
	affected, err := engine.ID(1).Delete(&user)
	fmt.Println(affected)
//等价于   DELETE FROM user Where id = ?
	affected, err := engine.Table("user").Where("name = ?", "lunny").Delete()
	fmt.Println(affected)
// 等价于 DELETE FROM user WHERE name = lunny 需要注意的是where 后起码有一个条件才可以
	user := models.User{}
	affected, err := engine.Where("name = ?", "sz").Delete(&user)
	fmt.Println(affected)
// 等价于 DELETE FROM user WHERE name = sz 需要注意的是where 后起码有一个条件才可以

```

查询数据

```go
//Query 最原始的也支持SQL语句查询，返回的结果类型为 []map[string][]byte。QueryString 返回 []map[string]string, //QueryInterface 返回 []map[string]interface{}.

	results1, err := engine.Query("select * from user")
	fmt.Println("results1", results1)
	results3, err := engine.QueryString("select * from user")
	fmt.Println("results3", results3)
	results5, err := engine.QueryInterface("select * from user")
	fmt.Println("results5", results5)

一般来说后面两个用的多一些 
	user := models.User{}
	has, err := engine.Get(&user)
	fmt.Println(has, user)
// 获取第一条数据 has 是一个bool类型
	users := models.User{}
	has, err := engine.Where("name = ?", "sz").Desc("id").Get(&users)
	fmt.Println(has, users)
// 需要注意的是get 只能得到单条数据 所以会取最后一条数据
	user := models.User{}
	var name string
	has, err := engine.Table(&user).Where("id = ?", 7).Cols("name").Get(&name)
	fmt.Println(has, name)
//等价于 SELECT name FROM user WHERE id = ? 查询单项的数据
	user := models.User{}
	var valuesMap = make(map[string]string)
	has, err := engine.Table(&user).Where("id = ?", 9).Get(&valuesMap)
	fmt.Println(has, valuesMap)
// 等价于 SELECT * FROM user WHERE id = ?  查询单项的数据
	user := models.User{}
	cols := []string{user.Name, user.Salt, user.Passwd}
	var valuesSlice = make([]interface{}, len(cols))
	has, err := engine.Table(&user).Where("id = ?", 8).Cols("name", "salt", "passwd").Get(&valuesSlice)
	fmt.Println(has, valuesSlice)
// 等价于 SELECT col1, col2, col3 FROM user WHERE id = ?

//查询多条数据
	var users []models.User
	engine.Where("name = ?", "sz").And("age > 10").Limit(10, 0).Find(&users)
	fmt.Println(users)
//等价于 SELECT * FROM user WHERE name = sz AND age > 10 limit 10 offset 0
// 当然可以使用Join和extends来组合使用
	type Detail struct {
		Id     int64
		UserId int64 `xorm:"index"`
	}

	type UserDetail struct {
		models.User `xorm:"extends"`
		Detail      `xorm:"extends"`
	}
	err = engine.Sync(new(Detail))
	var users []UserDetail
	err = engine.Table("user").Select("user.*, detail.*").
		Join("INNER", "detail", "detail.user_id = user.id").
		Where("user.name = ?", "sz").Limit(10, 0).
		Find(&users)
	fmt.Println(err, users)
// 等价于 SELECT user.*, detail.* FROM user INNER JOIN detail WHERE user.name = ? limit 10 offset 0
// 需要注意的是 需要先声明一下表的数据
```

#### 5 xorm 的其他操作

- `Count` 获取记录条数

```
	user := models.User{}
	counts, err := engine.Count(&user)
	fmt.Println(counts)
```

- `Sum` 求和函数

```
	user := models.User{}
	sumFloat64Slice, err := engine.Sums(&user, "age")
	fmt.Println(sumFloat64Slice)
```

- `Iterate` 和 `Rows` 根据条件遍历数据库，可以有两种方式: Iterate and Rows

```
	err = engine.Iterate(&models.User{Name: "sz"}, func(idx int, bean interface{}) error {
		user := bean.(*models.User)
		fmt.Println(user)
		return nil
	})
	fmt.Println(err)
```

```
	err = engine.BufferSize(100).Iterate(&models.User{Name: "sz"}, func(idx int, bean interface{}) error {
		user := bean.(*models.User)
		fmt.Println(user)
		return nil
	})
	fmt.Println(err)
```

```
	rows, err := engine.Rows(&models.User{Name: "sz"})
	// SELECT * FROM user
	defer rows.Close()
	beans := new(models.User)
	for rows.Next() {
		err = rows.Scan(beans)
		fmt.Println(beans)
	}

```

无事务DEMO

````GO
// 如下面的方法一样 如果没有事务的话执行到哪一步报错了前面的也会去执行
func mysqlDemo() error{
	dataSourceName := fmt.Sprintf("%s:%s@tcp(%s:%d)/%s?charset=%s", username, password, ipAddress, port, dbName, charset)
	engine, err := xorm.NewEngine("mysql", dataSourceName)
	if err != nil {
		fmt.Println(err)
	}
	session := engine.NewSession()
	defer session.Close()

	user1 := models.User{Name: "xiaoxiao", Age: 16, Created: time.Now()}
	if _, err := session.Insert(&user1); err != nil {
		return err
	}

	user2 := models.User{Name: "yyy"}
	if _, err := session.Where("id = ?", 2).Update(&user2); err != nil {
		return err
	}

	if _, err := session.Exec("delete from user where name = ?", user2.Name); err != nil {
		return err
	}

	return nil
}

func main() {

	err := mysqlDemo()
	fmt.Println(err)
}

````

有事务的DEMO

```
func mysqldemo2()  error{
	dataSourceName := fmt.Sprintf("%s:%s@tcp(%s:%d)/%s?charset=%s", username, password, ipAddress, port, dbName, charset)
	engine, err := xorm.NewEngine("mysql", dataSourceName)
	if err != nil {
		fmt.Println(err)
	}
	session := engine.NewSession()
	defer session.Close()

	session.Begin()
	defer func() {
		err:= recover()
		if err != nil{
			session.Rollback()
		}else{
			session.Commit()
		}
	}()

	user1 := models.User{Name: "xiaoxiao", Age: 16, Created: time.Now()}
	if _, err := session.Insert(&user1); err != nil {
		return err
	}

	user2 := models.User{Name: "yyy"}
	if _, err := session.Where("id = ?", 2).Update(&user2); err != nil {
		return err
	}

	if _, err := session.Exec("delete from user where name = ?", user2.Name); err != nil {
		return err
	}

	// add Commit() after all actions
	return session.Commit()
}


```

其可以简写为

```
func mysqldemo3() (bool, error) {
	dataSourceName := fmt.Sprintf("%s:%s@tcp(%s:%d)/%s?charset=%s", username, password, ipAddress, port, dbName, charset)
	engine, err := xorm.NewEngine("mysql", dataSourceName)
	if err != nil {
		fmt.Println(err)
	}
	res, err := engine.Transaction(func(session *xorm.Session) (interface{}, error) {
		user1 := models.User{Name: "xiaoxiao", Age: 16, Created: time.Now()}
		if _, err := session.Insert(&user1); err != nil {
			return false, err
		}

		user2 := models.User{Name: "yyy"}
		if _, err := session.Where("id = ?", 2).Update(&user2); err != nil {
			return false, err
		}

		if _, err := session.Exec("delete from user where name = ?", user2.Name); err != nil {
			return false, err
		}
		return true, nil
	})
	return res.(bool), err
}
```



