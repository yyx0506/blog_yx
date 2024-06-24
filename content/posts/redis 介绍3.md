---
title: "redis 持久化"
date: 2022-06-05
categories:
- tranquilpeak
- features
tags:
- redis
# keywords:
# - redis

thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->

### redis 介绍3

```
Redis是内存数据库，宕机后数据会消失。
Redis重启后快速恢复数据，要提供持久化机制
Redis持久化是为了快速的恢复数据而不是为了存储数据
```

Redis有两种持久化方式：RDB和AOF

```
在系统启动时，从这个完整的数据源中将数据load到Redis中
数据量较小，不易改变，比如：字典库（xml、Table）
通过info命令可以查看关于持久化的信息
```

#### RDB

RDB（Redis DataBase），是redis默认的存储方式，RDB方式是通过**快照**（ snapshotting ）完成

的。这一刻的数据, 不关注过程。

**触发快照的方式**

```
1. 符合自定义配置的快照规则
2. 执行save或者bgsave命令
3. 执行flushall命令
4. 执行主从复制操作 (第一次)
客户端就是bgsave 直接执行
```

#### AOF

```
AOF（append only file）是Redis的另一种持久化方式。Redis默认情况下是不开启的。开启AOF持久
化后
Redis 将所有对数据库进行过写入的命令（及其参数）（RESP）记录到 AOF 文件， 以此达到记录数据
库状态的目的，
这样当Redis重启后只要按顺序回放这些命令就会恢复到原始状态了。
AOF会记录过程，RDB只管结果

配置 redis.conf 
# 可以通过修改redis.conf配置文件中的appendonly参数开启 
appendonly yes 

# AOF文件的保存位置和RDB文件的位置相同，都是通过dir参数设置的。 
dir ./ 

# 默认的文件名是appendonly.aof，可以通过appendfilename参数修改 
appendfilename appendonly.aof
```

```
AOF 原理
AOF文件中存储的是redis的命令，同步命令到 AOF 文件的整个过程可以分为三个阶段：
命令传播：Redis 将执行完的命令、命令的参数、命令的参数个数等信息发送到 AOF 程序中。
缓存追加：AOF 程序根据接收到的命令数据，将命令转换为网络通讯协议的格式，然后将协议内容追加
到服务器的 AOF 缓存中。
文件写入和保存：AOF 缓存中的内容被写入到 AOF 文件末尾，如果设定的 AOF 保存条件被满足的话，
fsync 函数或者 fdatasync 函数会被调用，将写入的内容真正地保存到磁盘中。
命令传播  ---- 
当一个 Redis 客户端需要执行命令时， 它通过网络连接， 将协议文本发送给 Redis 服务器。服务器在
接到客户端的请求之后， 它会根据协议文本的内容， 选择适当的命令函数， 并将各个参数从字符串文
本转换为 Redis 字符串对象（ StringObject ）。每当命令函数成功执行之后， 命令参数都会被传播到
AOF 程序。
缓存追加 ---- 
当命令被传播到 AOF 程序之后， 程序会根据命令以及命令的参数， 将命令从字符串对象转换回原来的
协议文本。协议文本生成之后， 它会被追加到 redis.h/redisServer 结构的 aof_buf 末尾。
redisServer 结构维持着 Redis 服务器的状态， aof_buf 域则保存着所有等待写入到 AOF 文件的协
议文本（RESP）。
文件写入和保存  ---- 
每当服务器常规任务函数被执行、 或者事件处理器被执行时， aof.c/flushAppendOnlyFile 函数都会被
调用， 这个函数执行以下两个工作：
WRITE：根据条件，将 aof_buf 中的缓存写入到 AOF 文件。
SAVE：根据条件，调用 fsync 或 fdatasync 函数，将 AOF 文件保存到磁盘中。
AOF 保存模式 -----
Redis 目前支持三种 AOF 保存模式，它们分别是：
AOF_FSYNC_NO ：不保存。
AOF_FSYNC_EVERYSEC ：每一秒钟保存一次。（默认）
AOF_FSYNC_ALWAYS ：每执行一个命令保存一次。（不推荐）
以下三个小节将分别讨论这三种保存模式。
不保存 --------
在这种模式下， 每次调用 flushAppendOnlyFile 函数， WRITE 都会被执行， 但 SAVE 会被略过。
在这种模式下， SAVE 只会在以下任意一种情况中被执行：
Redis 被关闭
AOF 功能被关闭
系统的写缓存被刷新（可能是缓存已经被写满，或者定期保存操作被执行）
这三种情况下的 SAVE 操作都会引起 Redis 主进程阻塞。
每一秒钟保存一次（推荐）  ---- 
在这种模式中， SAVE 原则上每隔一秒钟就会执行一次， 因为 SAVE 操作是由后台子线程（fork）调用
的， 所以它不会引起服务器主进程阻塞。
每执行一个命令保存一次 --- 
在这种模式下，每次执行完一个命令之后， WRITE 和 SAVE 都会被执行。
另外，因为 SAVE 是由 Redis 主进程执行的，所以在 SAVE 执行期间，主进程会被阻塞，不能接受命令
请求。
AOF 保存模式对性能和安全性的影响
对于三种 AOF 保存模式， 它们对服务器主进程的阻塞情况如下：
```

```
AOF重写、触发方式、混合持久化
AOF记录数据的变化过程，越来越大，需要重写“瘦身”
Redis可以在 AOF体积变得过大时，自动地在后台（Fork子进程）对 AOF进行重写。重写后的新 AOF文
件包含了恢复当前数据集所需的最小命令集合。 所谓的“重写”其实是一个有歧义的词语， 实际上，
AOF 重写并不需要对原有的 AOF 文件进行任何写入和读取， 它针对的是数据库中键的当前值
Redis 不希望 AOF 重写造成服务器无法处理请求， 所以 Redis 决定将 AOF 重写程序放到（后台）子进
程里执行， 这样处理的最大好处是：
1、子进程进行 AOF 重写期间，主进程可以继续处理命令请求。
2、子进程带有主进程的数据副本，使用子进程而不是线程，可以在避免锁的情况下，保证数据的安全
性
不过， 使用子进程也有一个问题需要解决： 因为子进程在进行 AOF 重写期间， 主进程还需要继续处理
命令， 而新的命令可能对现有的数据进行修改， 这会让当前数据库的数据和重写后的 AOF 文件中的数
据不一致。
为了解决这个问题， Redis 增加了一个 AOF 重写缓存， 这个缓存在 fork 出子进程之后开始启用，
Redis 主进程在接到新的写命令之后， 除了会将这个写命令的协议内容追加到现有的 AOF 文件之外，
还会追加到这个缓存中。

redis.conf 配置
# 表示当前aof文件大小超过上一次aof文件大小的百分之多少的时候会进行重写。如果之前没有重写过， 
以启动时aof文件大小为准 
auto-aof-rewrite-percentage 100 
# 限制允许重写最小aof文件大小，也就是文件大小小于64mb的时候，不需要进行优化 
auto-aof-rewrite-min-size 64mb
```



```
混合持久化
RDB和AOF各有优缺点，Redis 4.0 开始支持 rdb 和 aof 的混合持久化。如果把混合持久化打开，aof
rewrite 的时候就直接把 rdb 的内容写到 aof 文件开头。
RDB的头+AOF的身体---->appendonly.aof
开启混合持久化
aof-use-rdb-preamble yes
```

```
RDB与AOF对比
1、RDB存某个时刻的数据快照，采用二进制压缩存储，AOF存操作命令，采用文本存储(混合) 2、RDB性能高、AOF性能较低
3、RDB在配置触发状态会丢失最后一次快照以后更改的所有数据，AOF设置为每秒保存一次，则最多
丢2秒的数据
4、Redis以主服务器模式运行，RDB不会保存过期键值对数据，Redis以从服务器模式运行，RDB会保
存过期键值对，当主服务器向从服务器同步时，再清空过期键值对。
AOF写入文件时，对过期的key会追加一条del命令，当执行AOF重写时，会忽略过期key和del命令。

内存数据库 rdb+aof 数据不容易丢 
有原始数据源： 每次启动时都从原始数据源中初始化 ，则 不用开启持久化 （数据量较小）
缓存服务器 rdb 一般 性能高

```

