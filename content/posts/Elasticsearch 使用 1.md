---
title: "Elasticsearch 基本操作"
date: 2022-05-15
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


### Elasticsearch 使用 1

#### 1.1 什么是Elasticsearch

```
Elasticsearch，简称为es， es是一个开源的高扩展的分布式全文检索引擎，它可以近乎实时的存储、检索数据；本身扩展性很好，可以扩展到上百台服务器，处理PB级别的数据。es也使用Java开发并使用Lucene作为其核心来实现所有索引和搜索的功能，但是它的目的是通过简单的RESTful API来隐藏Lucene的复杂性，从而让全文搜索变得简单。
```

#### 1.2 为什么使用es?

```
系统中的数据， 随着业务的发展，时间的推移， 将会非常多， 而业务中往往采用模糊查询进行数据的搜索， 而模糊查询会导致查询引擎放弃索引，导致系统查询数据时都是全表扫描，在百万级别的数据库中，查询效率是非常低下的，而我们使用 ES 做一个全文索引，将经常查询的系统功能的某些字段,比如某个词，某个指数之类的，说白了就是以空间换取时间（ES的内存占用十分巨大）
```

#### 1.3 es的特点和优势

```
    分布式实时文件存储，可将每一个字段存入索引，使其可以被检索到。 

    近乎实时分析的分布式搜索引擎。 

    分布式：索引分拆成多个分片，每个分片可有零个或多个副本。集群中的每个数据节点都可承载一个或多个分片，并且协调和处理各种操作； 

    负载再平衡和路由在大多数情况下自动完成。 

    可以扩展到上百台服务器，处理PB级别的结构化或非结构化数据（官网是这么说的）。也可以运行在单台PC上（已测试）。 

    支持插件机制，分词插件、同步插件、Hadoop插件、可视化插件等。
```



#### 2.1 es的基本操作（python）

第一步我们先直接安装elasticsearch (注意 我们在服务器上安装装的elasticsearch 是7.6.2版本的所以我们安装 的 python 版本 的 elasticsearch 也要是7版本 elasticsearch每个版本的api变动还是比较频繁的 )

Pip3 install elasticsearch == 7.13.0

#### 首先先创建一个index 

```
from elasticsearch import Elasticsearch
from datetime import datetime
# es = Elasticsearch([{'host': '10.1.26.59', 'port': 9200}])
es = Elasticsearch('http://121.5.58.115:9200/')

doc = {
    'author': 'kimchy',
    'text': 'Elasticsearch: cool. bonsai cool.',
    'timestamp': datetime.now(),
}
resp = es.index(index="test_index", id=1, body=doc)

```

然后我们去查询可以发现 已经有了此条数据

![](/img/py1es1.png)




然后我们执行下一条命令 不指定id 

```
doc = {
    'author': 'yyx',
    'text': 'test elasticsearch',
    'timestamp': datetime.now(),
}

resp = es.index(index="test_index",  body=doc)
```

再去es中查询结果，我们发现它并没有自增id为2 而是 直接生成了一个随机的id 


![](/img/py1es2.png)


我们直接使用python 去对es进行查询

```
res = es.get(index='test_index',id = 1)
print(res)
{'_index': 'test_index', '_type': '_doc', '_id': '1', '_version': 1, '_seq_no': 0, '_primary_term': 1, 'found': True, '_source': {'author': 'kimchy', 'text': 'Elasticsearch: cool. bonsai cool.', 'timestamp': '2022-05-03T12:50:40.143640'}}

```

得到的结果为如下值 会发现 es会将结构放在_source下，然后外层会有多个key进行修饰  found 代表找到与否



然后我们执行删除操作 将此条数据进行删除（elasticsearch 只能通过代码删除，不能在elasticsearch head操作）

```
es.delete(index='test_index',id='sD1QKIEBnZY8SBgf8fba' )
```

### es 的search操作



为了测试接下来的查询 我首先插入了几条数据 用于显示 如图
![](/img/py1es3.png)


tips : es 的查询检索方式称为DSL语句查询 

#### 1：term过滤

我们查询一下 author =yyx 的信息 其中 query 和bool 是固定写法, 第三个 must 是可变的 （例如 not must should 等后面都是要跟列表，列表里面的每一个字典都代表着一个条件

```
result = es.search(
    index="test_index",
    body={
        "query":{
            "bool":{
                "must":[{
                    "term":{
                        "author":"yyx"
                    }
                }]
            }
        }
    }
)
{'took': 2, 'timed_out': False, '_shards': {'total': 1, 'successful': 1, 'skipped': 0, 'failed': 0}, 'hits': {'total': {'value': 5, 'relation': 'eq'}, 'max_score': 0.24116206, 'hits': [{'_index': 'test_index', '_type': '_doc', '_id': '2', '_score': 0.24116206, '_source': {'author': 'yyx', 'test': 'practise', 'text': 'test elasticsearch   use term'}}, {'_index': 'test_index', '_type': '_doc', '_id': '3', '_score': 0.24116206, '_source': {'author': 'yyx', 'test': 'practise', 'text': 'test elasticsearch   use terms'}}, {'_index': 'test_index', '_type': '_doc', '_id': '4', '_score': 0.24116206, '_source': {'author': 'yyx', 'test': 'practise', 'age': 23, 'text': 'test elasticsearch   use range'}}, {'_index': 'test_index', '_type': '_doc', '_id': '5', '_score': 0.24116206, '_source': {'author': 'yyx', 'test': 'practise', 'age': 28, 'text': 'test elasticsearch   use range'}}, {'_index': 'test_index', '_type': '_doc', '_id': '6', '_score': 0.24116206, '_source': {'author': 'yyx', 'test': 'practise', 'age': 33, 'text': 'test elasticsearch   use range'}}]}}


最后得到的结果就是这样的一个结果 如果结果不存在的话会是['hits']['hits']的值为空 

```

#### 2 :  terms过滤

```
terms跟term有点类似，但terms允许指定多个匹配条件。如果某个字段指定了多个值，那么文档需要一起去做匹配：
{"terms":{"author":["yyx","kimchy"]}} 替换掉查询的语句块得到的结果
{'took': 13, 'timed_out': False, '_shards': {'total': 1, 'successful': 1, 'skipped': 0, 'failed': 0}, 'hits': {'total': {'value': 6, 'relation': 'eq'}, 'max_score': 1.0, 'hits': [{'_index': 'test_index', '_type': '_doc', '_id': '2', '_score': 1.0, '_source': {'author': 'yyx', 'test': 'practise', 'text': 'test elasticsearch   use term'}}, {'_index': 'test_index', '_type': '_doc', '_id': '3', '_score': 1.0, '_source': {'author': 'yyx', 'test': 'practise', 'text': 'test elasticsearch   use terms'}}, {'_index': 'test_index', '_type': '_doc', '_id': '1', '_score': 1.0, '_source': {'author': 'kimchy', 'text': 'Elasticsearch: cool. bonsai cool.', 'timestamp': '2022-06-03T12:50:40.143640'}}, {'_index': 'test_index', '_type': '_doc', '_id': '4', '_score': 1.0, '_source': {'author': 'yyx', 'test': 'practise', 'age': 23, 'text': 'test elasticsearch   use range'}}, {'_index': 'test_index', '_type': '_doc', '_id': '5', '_score': 1.0, '_source': {'author': 'yyx', 'test': 'practise', 'age': 28, 'text': 'test elasticsearch   use range'}}, {'_index': 'test_index', '_type': '_doc', '_id': '6', '_score': 1.0, '_source': {'author': 'yyx', 'test': 'practise', 'age': 33, 'text': 'test elasticsearch   use range'}}]}}
将author 是yyx 和 kimchy 都筛选了出来

```

#### 3:range 过滤 （只针对数字）

```
{"range":{"age":{"gte":20,"lt":30}}} 替换掉查询的语句块得到的结果
{'took': 5, 'timed_out': False, '_shards': {'total': 1, 'successful': 1, 'skipped': 0, 'failed': 0}, 'hits': {'total': {'value': 2, 'relation': 'eq'}, 'max_score': 1.0, 'hits': [{'_index': 'test_index', '_type': '_doc', '_id': '4', '_score': 1.0, '_source': {'author': 'yyx', 'test': 'practise', 'age': 23, 'text': 'test elasticsearch   use range'}}, {'_index': 'test_index', '_type': '_doc', '_id': '5', '_score': 1.0, '_source': {'author': 'yyx', 'test': 'practise', 'age': 28, 'text': 'test elasticsearch   use range'}}]}}
得到了 年龄在20 -30 的数据
范围操作符包含：
gt	::	大于
gte	::	大于等于
lt	::	小于
lte	::	小于等于
```

#### 4: exists和missing过滤

```
exists和missing过滤可以用于查找文档中是否包含指定字段或没有某个字段，类似于SQL语句中的IS_NULL条件
目前官方已经将 missing 这个api 删除了
{"exists":{"field":"test"}}
{'took': 3, 'timed_out': False, '_shards': {'total': 1, 'successful': 1, 'skipped': 0, 'failed': 0}, 'hits': {'total': {'value': 5, 'relation': 'eq'}, 'max_score': 1.0, 'hits': [{'_index': 'test_index', '_type': '_doc', '_id': '2', '_score': 1.0, '_source': {'author': 'yyx', 'test': 'practise', 'text': 'test elasticsearch   use term'}}, {'_index': 'test_index', '_type': '_doc', '_id': '3', '_score': 1.0, '_source': {'author': 'yyx', 'test': 'practise', 'text': 'test elasticsearch   use terms'}}, {'_index': 'test_index', '_type': '_doc', '_id': '4', '_score': 1.0, '_source': {'author': 'yyx', 'test': 'practise', 'age': 23, 'text': 'test elasticsearch   use range'}}, {'_index': 'test_index', '_type': '_doc', '_id': '5', '_score': 1.0, '_source': {'author': 'yyx', 'test': 'practise', 'age': 28, 'text': 'test elasticsearch   use range'}}, {'_index': 'test_index', '_type': '_doc', '_id': '6', '_score': 1.0, '_source': {'author': 'yyx', 'test': 'practise', 'age': 33, 'text': 'test elasticsearch   use range'}}]}
这个过滤只是针对已经查出一批数据来，但是想区分出某个字段是否存在的时候使用。
```

#### 5 : bool 过滤

```
bool过滤可以用来合并多个过滤条件查询结果的布尔逻辑，它包含一下操作符：

must		::	多个查询条件的完全匹配,相当于	and	。
must_not	::	多个查询条件的相反匹配，相当于	not	。
should		::	至少有一个查询条件匹配,相当于	or	。
这些参数可以分别继承一个过滤条件或者一个过滤条件的数组：
就像第一个语法一样
```

#### 6: match_all 查询

```
使用match_all可以查询到所有文档，是没有查询条件下的默认语句。

{"match_all":{}}

此查询常用于合并过滤条件。比如说你需要检索所有的邮箱,所有的文档相关性都是相同的，所以得到的_score为1
```

#### 7: multi_match查询

```
multi_match	查询允许你做match查询的基础上同时搜索多个字段：

{"multi_match":	{"query":"full text	search","fields":["title","body"]}}
```

#### 8: bool 查询

```
bool查询与bool过滤相似，用于合并多个查询子句。不同的是，bool过滤可以直接给出是否匹配成功，而bool查询要计算每一个查询子句的_score（相关性分值）。

must	 ::	查询指定文档一定要被包含。
must_not ::	查询指定文档一定不要被包含。
should	 ::	查询指定文档，有则可以为文档相关性加分。

以下查询将会找到title字段中包含"how to make millions"，并且"tag"字段没有被标为spam	。如果有标识为"starred"或者发布日期为2014年之前，那么这些匹配的文档将比同类网站等级高：

{"bool":{"must":{"match":{"title":"how to make millions"}},"must_not":{"match":{"tag":"spam"}},"should":[{"match":{"tag":"starred"}},{"range":{"date":{"gte":"2022-05-01"}}}]}}

提示：	如果bool查询下没有must子句，那至少应该有一个should子句。但是如果有must子句，那么没有should子句也可以进行查询。
```



### 本篇到此结束，下一篇我会介绍一下python 异步连接 elasticsearch

