---
title: "Elasticsearch 索引 以及 mapping"
date: 2022-05-17
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


### Elasticsearch 使用3

以下操作都基于kibana 操作 

#### 1: ES倒排索引

```
概述：倒排索引是全文检索的根基，理解了倒排索引之后才能算是入门了全文检索领域。倒排索引的的概念很简单，也很好
理解。Elasticsearch/Lucene是如何实现这个结构的呢？
在7.0以前使用的是TF-IDF算法 7.0 以后是BM25算法
倒排索引由两部分组成，所有独立的词列表称为索引，词对应的一系列表统称为倒排表。
索引表，叫 Terms Dictionary ，是由于一系列的Term组成的。
倒排表，称 Postings List ，即是由所有的Term对应的Postings组成的。
所有的索引都是以数据结构为载体，以文件形式落地

```

#### 2:索引的CURD

```
1 创建索引 ：PUT /索引名?pretty
2 查询索引 ：GET /索引名/_search
3 删除索引 ：DELETE /index?pretty
4 插入数据 ：PUT /索引名/_doc/id {Json 数据} 
其中 put 和 post 都可以用作 更新操作 二者的区别就是 put是全量替换 而 post 可以 指定字段更新
如图所示
```

#### 3: Mapping- 映射

```
概念：映射是定义了文档及包含的字段的存储和索引方式的过程。
分为两种映射方式 dynamic mapping (动态映射或自动映射)  expllcit mapping(静态映射或手工映射或显示映射)
mapping是处理数据的方式和规则方面做一些限制，如某个字段的数据类型、默认值、分析器、是否被索引等等，这
些都是映射里面可以设置的，其它就是处理es里面数据的一些使用规则设置也叫做映射，按着最优规则处理数据对性
能提高很大，因此才需要建立映射，并且需要思考如何建立映射才能对性能更好。
当不指定是就是动态映射 而 我们第二篇使用时 用了 data 就是使用的 静态映射

```

##### 如何查看mapping

```
GET /nana_index/_mapping 
```

##### 两种映射

Dynamic field mapping :

整数 （long） 浮点数（float） true|false (boolean) 日期（date）数组（取决于数组中第一个有效值）对象(object) 

字符串（如果不是数字和日期类型，那会被映射成text 和 keyword 两个类型 ）

Tips : 除了上述字段类型之外，其他类型都必须显示映射，也就是必须手工指定，因为其他类型es无法自动识别 

Explicit field mapping :手动映射 在使用二内就已经用过了 这个手工映射索引 创建以后类型是无法修改的

```
PUT index_test
{
  "mappings": {
    "properties": {
      "field1": {
        "type": "text"
      }
    }
  }
}
```

##### 映射参数

```
1  index :是否对创建当前字段创建倒排索引，默认是true 如果不创建索引，该字段不会通过索引被搜索到，但是仍然会在source元数据中展示
2  analyzer :指定分词器
3  boost: 对当前字段相关度的评分权重，默认1
4  coerce: 是否允许强制类型转换 true"1"=>1 false"1"=<1
5  copy_to : 该参数允许将多个字段的值复制到组字段中，然后可以将其作为单个字段进行查询
6  doc_values: 为了提升排序和聚合效率，默认true ，如果确定不需要对字段进行排序或聚合，也不需要通过脚本访问字段值，则可以禁用doc值以节省磁盘空间（不支持text 和 annotated_text）
7  dynamic :控制是否可以动态添加新字段 默认是true 
8  eager_global_ordinals :用于聚合的字段上，优化聚合性能
9  enable : 是否创建倒排索引，也可以对字段操作
10 format : 格式化
还有其他的参数就不一一赘述了
```

#### 4: 文档的CURD

```
1）新增

    # 新增单条数据，并指定es的id 为 1
    PUT nana_index/_doc/1
    {
    "name": "Te Hero"
    }
    # 新增单条数据，使用ES自动生成id
    POST nana_index/_doc
    {
    "name": "Te Hero2"
    }

    # 使用 op_type 属性，强制执行某种操作
    PUT nana_index/_doc/1?op_type=create
    {
        "name": "Te Hero3"
    }
    注意：op_type=create强制执行时，若id已存在，ES会报“version_conflict_engine_exception”。
    op_type 属性在实践中同步数据时是有用的，后面讲解数据库与ES的数据同步问题时，nana再为大家详细讲解。
    我们查询数据，看下效果：GET nana_index/_search

    {
    "took": 1,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": 2,
        "max_score": 1,
        "hits": [
        {
            "_index": "nana_index",
            "_type": "_doc",
            "_id": "1",
            "_score": 1,
            "_source": {
            "name": "Te Hero"
            }
        },
        {
            "_index": "nana_index",
            "_type": "_doc",
            "_id": "P7-FCHIBJxE1TMY0WNGN",
            "_score": 1,
            "_source": {
            "name": "Te Hero2"
            }
        }
        ]
    }
    }
2）修改

    # 根据id，修改单条数据
    （ps：修改语句和新增语句相同，可以理解为根据ID，存在则更新；不存在则新增）
    PUT nana_index/_doc/1
    {
    "name": "Te Hero-update"
    }

    # 根据查询条件id=10，修改name="更新后的name"（版本冲突而不会导致_update_by_query 中止）
    POST nana_index/_update_by_query
    {
    "script": {
        "source": "ctx._source.name = params.name",
        "lang": "painless",
        "params":{
        "name":"更新后的name"
        }
    },
    "query": {
        "term": {
        "id": "10"
        }
    }
    }

    关于文档的更新，Update By Query API，对于该API的使用，将其归类为进阶知识，后续章节将为大家更深入的讲解。

3）查询

    # 1、根据id，获取单个数据
    GET nana_index/_doc/1
    结果：
    {
    "_index": "nana_index",
    "_type": "_doc",
    "_id": "1",
    "_version": 5,
    "found": true,
    "_source": {
        "name": "Te Hero-update",
        "age": 18
    }
    }

    # 2、获取索引下的所有数据
    GET nana_index/_doc/_search
    结果：
    {
    "took": 1,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": 3,
        "max_score": 1,
        "hits": [
        {
            "_index": "nana_index",
            "_type": "_doc",
            "_id": "P7-FCHIBJxE1TMY0WNGN",
            "_score": 1,
            "_source": {
            "name": "Te Hero2"
            }
        },
        {
            "_index": "nana_index",
            "_type": "_doc",
            "_id": "_update",
            "_score": 1,
            "_source": {
            "name": "Te Hero3"
            }
        },
        {
            "_index": "nana_index",
            "_type": "_doc",
            "_id": "1",
            "_score": 1,
            "_source": {
            "name": "Te Hero-update",
            "age": 18
            }
        }
        ]
    }
    }

    # 3、条件查询（下一节详细介绍）
    GET nana_index/_doc/_search
    {
    "query": {
        "match": {
        "name": "2"
        }
    }
    }
    结果：
    {
    "took": 1,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": 1,
        "max_score": 0.9808292,
        "hits": [
        {
            "_index": "nana_index",
            "_type": "_doc",
            "_id": "P7-FCHIBJxE1TMY0WNGN",
            "_score": 0.9808292,
            "_source": {
            "name": "Te Hero2"
            }
        }
        ]
    }
    }

4）删除

    # 1、根据id，删除单个数据
    DELETE nana_index/_doc/1

    # 2、delete by query
    POST nana_index/_delete_by_query
    {
    "query": {
        "match": {
        "name": "2"
        }
    }
    }
```

#### 5: 批量操作 bulk API

```
# 批量操作

        POST _bulk
        { "index" : { "_index" : "nana_test1", "_type" : "_doc", "_id" : "1" } }
        { "this_is_field1" : "this_is_index_value" }
        { "delete" : { "_index" : "nana_test1", "_type" : "_doc", "_id" : "2" } }
        { "create" : { "_index" : "nana_test1", "_type" : "_doc", "_id" : "3" } }
        { "this_is_field3" : "this_is_create_value" }
        { "update" : {"_id" : "1", "_type" : "_doc", "_index" : "nana_test1"} }
        { "doc" : {"this_is_field2" : "this_is_update_value"} }

# 查询所有数据
        GET nana_test1/_doc/_search
        结果：
        {
        "took": 33,
        "timed_out": false,
        "_shards": {
            "total": 5,
            "successful": 5,
            "skipped": 0,
            "failed": 0
        },
        "hits": {
            "total": 2,
            "max_score": 1,
            "hits": [
            {
                "_index": "nana_test1",
                "_type": "_doc",
                "_id": "1",
                "_score": 1,
                "_source": {
                "this_is_field1": "this_is_index_value",
                "this_is_field2": "this_is_update_value"
                }
            },
            {
                "_index": "nana_test1",
                "_type": "_doc",
                "_id": "3",
                "_score": 1,
                "_source": {
                "this_is_field3": "this_is_create_value"
                }
            }
            ]
        }
        }
```

