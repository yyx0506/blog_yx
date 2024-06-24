
---
title: "ES 原理剖析"
date: 2022-05-25
categories:
- tranquilpeak
- features
tags:
- Elasticsearch
# keywords:
# - Elasticsearch
thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->

### ES 原理剖析 



#### 1 ：读写流程解析

##### 1.1 创建文档

向ES中添加一个文档对象，由于ES是分布式集群并且底层设计为一个索引有众多shard(分片)，所以添加文档时需要

确定该文档属于哪个分片，确定的规则为(当分片为空时集群就会不健康，但是不影响使用)：

```
shard = hash(routing) % number_of_primary_shards
routing是一个可变值，默认是文档的_id，也可以设置成一个自定义的值
routing通过hash函数生成一个数字，然后这个数字再除以number_of_primary_shards（主分片的数量）后得
到余数 。这个分布在0到number_of_primary_shards - 1之间的余数，就是我们所寻求的文档所在分片的位
置。
```

##### 1.2 写文档流程

以官网的例子进行分析，从图中能看出一个集群由三个节点组成，有两个分片，两个副本。

![](/img/esprinciple1.png)

写操作必须在主分片上面完成之后，才能被复制到其他节点作为分片副本，新建、索引和删除请求都是写操作。

```
客户端向master发送写入请求，该节点作为协调节点；
根据文档的_id确定分片,图中请求文档属于分片0，协调节点请求转到节点的主分片；
在节点3上执行请求，成功之后，节点3根据副本数将请求并行转到副本分片对应节点，一旦副本分片执行完成，都向节点3报告成功，节点3将向协调节点报告成功，协调节点再向客户端报告成功。
```

客户端收到成功响应时，则变更操作是安全的 

##### 1.3读文档流程

```
一个搜索请求必须询问请求的索引中所有分片的某个副本来进行匹配。假设一个索引有5个主分片，每个主分片有一
个副本分片，共10分片，一次搜索请求会由5个分片来共同完成，他们可能是主分片，也可能是副分片。也就是说，
一次搜索请求只会命中所有分片副本中的一个。
一次检索流程主要分为两个阶段：
```

Query阶段 如图所示
![](/img/esprinciple2.png)


```
1. 客户端发送search请求到NODE3。 2. NODE3将查询请求转发到索引的每个主分片或副分片中。
3. 每个分片在本地执行查询，并使用本地的Term/Docuemnt Frequency信息进行打分，添加结果到大小为
from+size的本地优先队列中。
4. 每个分片返回各自优先队列中所有文档的ID和排序值给协调节点，协调节点合并这些值到自己的优先队列中，
产生一个全局排序后的列表。
```

Fetch阶段 如图所示

![](/img/esprinciple3.png)

```
1. 协调节点向相关NODE发送GET请求。
2. 分片所在节点向协调节点返回数据。
3. 协调节点等待所有文档被取得，然后返回给客户端。
分片所在节点在返回文档数据时，处理有可能出现的 _source字段和高亮参数
```



##### 1.4 实时搜索

es 写操作流程，当一个写请求发送到 es 后，es 将数据写入 memory buffer 中，并添加事务日志（ translog ）。

如果每次一条数据写入内存后立即写到硬盘文件上，由于写入的数据肯定是离散的，因此写入硬盘的操作也就是随机

写入了。硬盘随机写入的效率相当低，会严重降低es的性能。因此 es 在设计时在 memory buffer 和硬盘间加入了 Linux 的高速缓存（ File system cache ）来提高 es 的写效

率。

当写请求发送到 es 后，es 将数据暂时写入 memory buffer 中，此时写入的数据还不能被查询到。默认设置下，es

每1秒钟将 memory buffer 中的数据 refresh 到 Linux 的 File system cache ，并清空 memory buffer ，此时

写入的数据就可以被查询到了

![](/img/esprinciple4.png)

refresh API

在 Elasticsearch 中，写入和打开一个新段的轻量的过程叫做 refresh 。 默认情况下每个分片会每秒自动刷新一次。

这就是为什么我们说 Elasticsearch 是 近 实时搜索： 文档的变化并不是立即对搜索可见，但会在一秒之内变为可

见。

这些行为可能会对新用户造成困惑： 他们索引了一个文档然后尝试搜索它，但却没有搜到。这个问题的解决办法是

用 refresh API 执行一次手动刷新：



##### 1.5 倒排索引

Elasticsearch 使用一种称为倒排索引的结构，它适用于快速的全文搜索。一个倒排索引由文档中所有不重复词的列

表构成，对于其中每个词，有一个包含它的文档列表。

例如，假设我们有两个文档，每个文档是如下内容：

```
1. The quick brown fox jumped over the lazy dog 
2. Quick brown foxes leap over lazy dogs in summer
```

要创建倒排索引，首先要将每个文档内容拆分成单独的词，创建一个包含所有不重复词条的排序列表，然后列出每个

词条出现在哪个文档。结果如下所示

```
Term Doc_1 Doc_2 ------------------------- 
Quick    |   | X 
The      | X | 
brown    | X | X 
dog      | X | 
dogs     |   | X 
fox      | X | 
foxes    |   | X 
in       |   | X 
jumped   | X | 
lazy     | X | X 
leap     |   | X 
over     | X | X 
quick    | X | 
summer   |   | X 
the      | X |
```

现在，如果我们想搜索 quick brown ，我们只需要查找包含每个词条的文档：

```
Term Doc_1 Doc_2 
------------------------- 
brown | X | X 
quick | X | 
------------------------ 
Total | 2 | 1
```

两个文档都匹配，但是第一个文档比第二个匹配度更高。如果我们使用仅计算匹配词条数量的简单 相似性算法，那

么，我们可以说，对于我们查询的相关性来讲，第一个文档比第二个文档更佳。