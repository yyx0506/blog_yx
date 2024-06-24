---
title: "Golang 单例应用的实际操作"
date: 2023-07-05
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

#### Golang 单例应用的实际操作

```


package main

import (
	"context"
	"fmt"
	"github.com/redis/go-redis/v9"
	"sync"
)

type RedisDBConfig struct {
	Host     string
	Port     string
	Password string
}

type RedisInstanceConfig struct {
	Name     string
	Instance *redis.Client
	DBs      map[int]*redis.Client
}

type RedisManager struct {
	instances sync.Map
	once      sync.Once
}

var (
	redisDBConfigs = map[string]RedisDBConfig{
		"instance1": {"localhost", "6379", ""}, // Redis实例1配置
		"instance2": {"localhost", "6380", ""}, // Redis实例2配置
		// 可以添加更多的Redis实例配置
	}
)

func NewRedisManager() *RedisManager {
	return &RedisManager{}
}

func (m *RedisManager) getRedisInstance(redisName string) (*RedisInstanceConfig, error) {
	instance, ok := m.instances.Load(redisName)
	if ok {
		return instance.(*RedisInstanceConfig), nil
	}

	m.once.Do(func() {
		for name := range redisDBConfigs {
			m.instances.Store(name, &RedisInstanceConfig{
				Name: name,
				DBs:  make(map[int]*redis.Client),
			})
		}
	})

	config, exists := redisDBConfigs[redisName]
	if !exists {
		return nil, fmt.Errorf("Redis instance '%s' not found", redisName)
	}

	address := fmt.Sprintf("%s:%s", config.Host, config.Port)
	opts := &redis.Options{
		Addr:     address,
		Password: config.Password,
	}

	instanceClient := redis.NewClient(opts)

	// Perform a simple ping to test the connection
	if err := instanceClient.Ping(context.Background()).Err(); err != nil {
		return nil, err
	}

	instanceConfig := &RedisInstanceConfig{
		Name:     redisName,
		Instance: instanceClient,
		DBs:      make(map[int]*redis.Client),
	}
	m.instances.Store(redisName, instanceConfig)

	return instanceConfig, nil
}

func (m *RedisManager) GetDB(redisName string, db int) (*redis.Client, error) {
	instance, err := m.getRedisInstance(redisName)
	if err != nil {
		return nil, err
	}

	dbClient, ok := instance.DBs[db]
	if !ok {
		dbClient = redis.NewClient(&redis.Options{
			Addr:     instance.Instance.Options().Addr,
			Password: instance.Instance.Options().Password,
			DB:       db,
		})
		instance.DBs[db] = dbClient
	}

	// Perform a simple ping to test the connection
	if err := dbClient.Ping(context.Background()).Err(); err != nil {
		return nil, err
	}

	return dbClient, nil
}
func main() {
	manager := NewRedisManager()

	// Example usage:
	redisName := "xjp_odds" // Set the desired redisName here ("instance1", "instance2", etc.)
	db := 0                 // Set the desired database number here

	dbClient, err := manager.GetDB(redisName, db)
	if err != nil {
		panic(err)
	}

	// You now have the Redis client instance for the specified Redis instance and database.
	// You can use `dbClient` to perform Redis operations on the specified database.
	fmt.Println("Connected to Redis instance:", redisName, "DB:", db)
	fmt.Println(dbClient)
}

解释一下这段代码：

1. 首先我们定义了两个结构体：
   - `RedisDBConfig`：表示一个Redis数据库的配置，包含主机、端口和密码等字段。
   - `RedisInstanceConfig`：表示一个Redis实例的配置，包含实例名、指向该实例的Redis客户端指针，以及一个映射将数据库和对应的Redis客户端关联起来。

2. 接着，我们创建了 `RedisManager` 结构体，负责管理单例Redis连接。它包含两个字段：
   - `instances`：一个 sync.Map，用于使用实例名作为键存储 `RedisInstanceConfig` 实例。
   - `once`：一个 `sync.Once` 变量，确保单例的初始化仅执行一次。

3. `NewRedisManager`：一个构造函数，用于创建新的 `RedisManager` 实例。它简单地返回一个 `RedisManager` 实例。

4. `getRedisInstance`：`RedisManager` 的私有方法，负责获取特定Redis实例的配置并创建相应的Redis客户端。它以 `redisName` 作为输入参数，从 `redisDBConfigs` 映射中查找配置，如果实例不存在，则为其创建一个新的 `RedisInstanceConfig`，同时为其初始化一个Redis客户端，并将其存储在 `instances` 映射中。

5. `GetDB`：`RedisManager` 的公共方法，用于获取特定Redis实例和数据库的Redis客户端。它以 `redisName` 和 `db` 作为输入参数，首先调用 `getRedisInstance` 方法以确保实例存在且其Redis客户端已初始化。然后，它检查是否已存在指定数据库的Redis客户端；如果不存在，则为该数据库创建一个新的Redis客户端，并将其存储在 `RedisInstanceConfig` 的映射中。

6. 在 `main` 函数中，我们创建了一个 `RedisManager` 实例。然后，我们调用 `GetDB` 方法来获取指定 `redisName` 和 `db` 的Redis客户端。这样，我们可以确保为每个Redis实例和数据库只创建一个Redis客户端，避免不必要的开销和可能的连接问题。

总体而言，这段代码提供了一种简洁的方式来管理Redis连接，并确保每个实例和数据库的Redis客户端仅被创建一次，从而避免不必要的重复连接和潜在的连接问题。 

关于出现的 once.do 算是 Golang 特有的  下面是一个简单的示例代码 

package main

import (
	"fmt"
	"sync"
)

func main() {
	var once sync.Once

	// 定义一个函数，我们希望它只执行一次
	doSomething := func() {
		fmt.Println("Doing something...")
	}

	// 使用 sync.Once 的 Do 方法，传入希望执行的函数
	// 无论调用多少次，doSomething 函数都只会被执行一次
	once.Do(doSomething)

	// 尝试多次调用 Do 方法，但函数 doSomething 只会执行一次
	once.Do(doSomething)
	once.Do(doSomething)
}

```

#### 2:Golang 单例mysql

```
package main

import (
	"database/sql"
	"encoding/json"
	"fmt"
	"io/ioutil"
	"sync"
	"time"

	_ "github.com/go-sql-driver/mysql"
)

type MySQLConfig struct {
	Host     string `json:"host"`
	Port     string `json:"port"`
	User     string `json:"user"`
	Password string `json:"password"`
	Database string `json:"database"`
}

type MySQLManager struct {
	instances map[string]*sql.DB
	configs   map[string]MySQLConfig
	once      sync.Once
}

var mySQLMgr *MySQLManager

func GetMySQLManager() *MySQLManager {
	once.Do(func() {
		mySQLMgr = &MySQLManager{
			instances: make(map[string]*sql.DB),
			configs:   make(map[string]MySQLConfig),
		}

		configData, err := ioutil.ReadFile("/Users/shou/Desktop/go_project/internal/mysql/mysql_config.json")
		if err != nil {
			panic("Failed to read MySQL config file")
		}

		var configs map[string]MySQLConfig
		err = json.Unmarshal(configData, &configs)
		if err != nil {
			panic("Failed to parse MySQL config")
		}
		fmt.Println(configs)
		mySQLMgr.configs = configs
	})

	return mySQLMgr
}

func (m *MySQLManager) GetDBInstance(dbName string) (*sql.DB, error) {
	// Check if the instance already exists in the map
	if db, ok := m.instances[dbName]; ok {
		return db, nil
	}

	// Get the configuration for the requested database
	config, ok := m.configs[dbName]
	if !ok {
		return nil, fmt.Errorf("database configuration not found for %s", dbName)
	}

	// Create a new database connection using the configuration
	db, err := sql.Open("mysql", fmt.Sprintf("%s:%s@tcp(%s:%s)/%s", config.User, config.Password, config.Host, config.Port, config.Database))
	if err != nil {
		return nil, err
	}

	// Store the instance in the map
	m.instances[dbName] = db
	return db, nil
}

// 使用 sync.Once 来保证只初始化一次
var once sync.Once

type Gift struct {
	id     int
	logo   string
	price  int
	name   string
	update time.Time
}

func main() {
	// Get the MySQLManager singleton instance
	manager := GetMySQLManager()

	// Get a MySQL database instance by name
	db1, err := manager.GetDBInstance("user_yx")
	if err != nil {
		fmt.Println("Error getting db1:", err)
		return
	}
	defer db1.Close()
	// 查询表结构
	rows, err := db1.Query("SELECT id, logo, price,name FROM ksgift")
	if err != nil {
		fmt.Println("Error querying table:", err)
		return
	}
	defer rows.Close()

	// 解析表结构
	var gifts []Gift

	for rows.Next() {
		var gift Gift
		err := rows.Scan(&gift.id, &gift.logo, &gift.price, &gift.name)
		if err != nil {
			fmt.Println("Error scanning row:", err)
			return
		}
		gifts = append(gifts, gift)
	}

	// 打印表结构
	for _, gift := range gifts {
		fmt.Println(gift.id, gift.name, gift.logo)
	}

	// Use the db1 instance for database operations
	// ...

	// Get another MySQL database instance by name
	//db2, err := manager.GetDBInstance("db2")
	//if err != nil {
	//	fmt.Println("Error getting db2:", err)
	//	return
	//}
	//defer db2.Close()

	// Use the db2 instance for database operations
	// ...
}

为什么 在倒入前 有一个 _  ？

在Go语言中，使用_符号导入包，表示只导入该包，但不直接使用包中的任何变量、函数或方法。这种方式通常用于导入包的初始化操作或者是为了执行包的init()函数，但不需要在代码中使用该包的其他功能。

在你的代码中，_ "github.com/go-sql-driver/mysql"是导入了github.com/go-sql-driver/mysql包，但由于没有在代码中直接使用该包的任何功能，所以加上了_符号，告诉编译器这是一个只导入但不直接使用的包。

实际上，这段代码中并没有直接使用github.com/go-sql-driver/mysql包中的任何功能，而是通过database/sql包的接口调用进行了数据库操作。因此，可以使用_来避免在代码中显式导入该包，而仅仅是为了让github.com/go-sql-driver/mysql包的init()函数被执行，从而注册MySQL数据库驱动。



这段代码是一个使用Golang实现的单例模式，用于管理多个MySQL数据库连接。它还包含了从数据库表中查询数据的示例。

代码逻辑如下：

1. 定义了MySQLConfig结构体，用于解析存储数据库配置的JSON文件。
2. 定义了MySQLManager结构体，其中包含了一个用于保存数据库连接实例的map，以及一个用于保存数据库配置的map，还有一个sync.Once来确保单例只被初始化一次。
3. GetMySQLManager函数返回MySQLManager单例实例，使用sync.Once确保只初始化一次，其中读取mysql_config.json配置文件，解析其中的配置，将其保存在MySQLManager的configs字段中。
4. GetDBInstance方法用于根据数据库名从MySQLManager中获取相应的数据库连接实例，如果实例不存在，则根据配置创建一个新的连接实例，并将其保存在instances字段中。
5. Gift结构体用于存储数据库表中的一条记录。
6. 在main函数中，先获取MySQLManager的单例实例，然后通过GetDBInstance方法获取数据库连接实例db1，之后使用db1执行查询表的操作，并将查询结果解析为Gift结构体，最后打印表结构。

请确保在运行代码之前，已经正确配置mysql_config.json文件，并包含了所需的数据库信息，以便能够成功连接到MySQL数据库。


我们将 mysql_config.json 可以单独作为一个配置文件来放置
{
  "user_yx": {
    "host": "xxxxxxxxx",
    "port": "3306",
    "user": "xxxxxxx",
    "password": "xxxxxx",
    "database": "xxxxxx"
  },
  "db2": {
    "host": "localhost",
    "port": "3306",
    "user": "user2",
    "password": "password2",
    "database": "database2"
  }

}

```

