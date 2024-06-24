---
title: "Elasticsearch python  aioelasticsearch 操作"
date: 2022-05-16
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

### Elasticsearch 使用2

上一篇文章我们讲述了python 的es基本操作,这一篇文章我们继续深入了解

首先开始我们在连接elasticsearch 就要使用 aioelasticsearch 了 异步连接

```
class AioESService:
    instances = {}
    es_config_production = {
        'test': DEVE_ES_URL,
    }
    es_config_development = { 
        'test': DEVE_ES_URL,
    }

    def __init__(self):
        pass

    @classmethod
    async def get(cls, name='test'):
        instance = cls.instances.get(name)
        if instance is None:
            config = cls.es_config_production[name] if AppCtx.get_env() == 'production' else cls.es_config_development[
                name]
            es = Elasticsearch(config, timeout=300)
            cls.instances[name] = instance = es
        return instance

    @classmethod
    async def close(cls):
        '''关闭连接
        '''
        instances = cls.instances
        for i in instances:
            if instances[i]:
                await instances[i].close()
        cls.instances = {}
```

具体的其他含义就不赘述了 其中 aioelasticsearch 是0.6.0版本的

然后我们写一个简单的查询语句 可以看到结果输出出来了

```
import asyncio

from asyncredis.aioes_service import AioESService

class AioEstest(object):

    async def test(self):
        es = await AioESService.get()
        query = {'bool': {'must': [{'terms':{"author":["yyx","kimchy"]}}]}}
        body = {'query': query }
        result = await es.search(index='test_index', body=body)
        print(result)

if __name__ == '__main__':
    asyncio.run(AioEstest().test())
    
{'took': 2, 'timed_out': False, '_shards': {'total': 1, 'successful': 1, 'skipped': 0, 'failed': 0}, 'hits': {'total': {'value': 6, 'relation': 'eq'}, 'max_score': 1.0, 'hits': [{'_index': 'test_index', '_type': '_doc', '_id': '2', '_score': 1.0, '_source': {'author': 'yyx', 'test': 'practise', 'text': 'test elasticsearch   use term'}}, {'_index': 'test_index', '_type': '_doc', '_id': '3', '_score': 1.0, '_source': {'author': 'yyx', 'test': 'practise', 'text': 'test elasticsearch   use terms'}}, {'_index': 'test_index', '_type': '_doc', '_id': '1', '_score': 1.0, '_source': {'author': 'kimchy', 'text': 'Elasticsearch: cool. bonsai cool.', 'timestamp': '2022-06-03T12:50:40.143640'}}, {'_index': 'test_index', '_type': '_doc', '_id': '4', '_score': 1.0, '_source': {'author': 'yyx', 'test': 'practise', 'age': 23, 'text': 'test elasticsearch   use range'}}, {'_index': 'test_index', '_type': '_doc', '_id': '5', '_score': 1.0, '_source': {'author': 'yyx', 'test': 'practise', 'age': 28, 'text': 'test elasticsearch   use range'}}, {'_index': 'test_index', '_type': '_doc', '_id': '6', '_score': 1.0, '_source': {'author': 'yyx', 'test': 'practise', 'age': 33, 'text': 'test elasticsearch   use range'}}]}}

```

再讲接下来的查询以及其他聚合高级的用法前，我们先补充一下es的类型。

String 类型

```
keyword text
```

数值类型

```
整数型：long、integer、short、byte
浮点型：float、half_float、scaled_float
```

时间类型

```
data
```

布尔类型

```
boolean
```

二进制类型

```
binary
```

区间类型

```
integer_range、long_range、float_range、double_range、date_range 就是都和range相关
```

复杂类型

```
数组类型 array
对象类型 object 用于单个json 对象
Nested类型 nested 用于json 对象数组
```

特定类型

```
GEO地址位置类型 Geo-point、Geo-shape
ip类型 ip
自动补全类型 completion
令牌计数数据类型 token_count
percolate类型 mumur3
父子索引 percolator
别名类型 alias
```

#### 1:String 类型 

```
String类型可以和java的string、mysql的varchar text等同，但是为何会分为text、keyword呢？这两者又有什么区别？text 和 keyword 的区别就在于是否分词,举个简单的例子，当你存入“最长的电影”这句话，如果使用了分词就会被存储为“最长的”、“电影”。如果使用的是text类型存储，如果我们不去指定分词器，那么ES就会使用默认的分词器 standard 而且选择text时就会默认创建了倒排索引

```

| 分词器              |                           说明                           |
| ------------------- | :------------------------------------------------------: |
| Standard Analyzer   |            默认分词器，按照词语切分，小写处理            |
| Simple Analyzer     |          按照非字母切分（符号被过滤），小写处理          |
| Stop Analyzer       |              小写处理，停用词过滤(the,a,is)              |
| Whitespace Analyzer |                  按照空格切分，不转小写                  |
| Keyword Analyzer    |                不分词，直接将输入当做输出                |
| Patter Analyzer     |             正则表达式，默认\W+(非字符分割)              |
| Language            |                      提供30多种默认                      |
| Customer Analyzer   |                       自定义分词器                       |
| IK                  |            支持自定义词库，支持热更新分词字典            |
| THULAC              | 清华大学自然语言处理和社会人文计算实验室的一套中文分词器 |

关于 es 的分词器我们可以简单做个了解

```
举例简单说明一下 ： “我爱你周杰伦”  tips es 遵守者restfulapi 所以每次执行都是一次请求
POST _analyze 

{
    "text": ["我爱你周杰伦"],
    "analyzer": "standard"
}
1.创建索引

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
2.存入数据

POST index_test/_doc/1
{
  "mappings": {
    "properties": {
      "field1": "我爱你周杰伦"
    }
  }
}


```

```
3.查询 索引下的所有数据

POST index_test/_search

4.条件查询，等价于mysql的 where field1 = "我爱你周杰伦" 。发现居然查询不到

POST nanatest/_search
{
  "query": {
    "term": {
      "mappings.properties.field1": {
        "value": "我爱你周杰伦"
      }
    }
  }
}
5.再根据刚才的ES分词效果，我们检索其中一个字，居然神奇的检索到了

POST nanatest/_search
{
  "query": {
    "term": {
      "mappings.properties.field1": {
        "value": "我"
      }
    }
  }
}

这是为什么呢？我们发现在使用term查询（等价于mysql的=）时却查不到结果，其实就是因为text类型会分词，简单理解就是“我爱你周杰伦”这句话在ES的倒排序索引中存储的是单个字，所以无法检索。
```

#### keyword——不会分词

### date 时间类型 —— 可规定格式

```
对于date类型，和mysql的几乎一样，唯一的注意点就是，储存的格式，ES是可以控制的。

PUT my_index

{
  "mappings": {
      "properties": {
        "date": {
          "type": "date"
        }
      }
  }
}

POST my_index/_doc/1
{
  "mappings": {
    "properties": {
      "date": "2015-01-01"
    }
  }
}
POST my_index/_doc/2
{
  "mappings": {
    "properties": {
      "date": "2015-01-01T12:10:30Z"
    }
  }
}

PUT my_index/_doc/3
{
  "mappings": {
    "properties": {
      "date": "1420070400001"
    }
  }
}

POST my_index/_search
{
  "sort": { "mappings.properties.date": "asc"}
}

大家可以用kibana试试，3种格式都可以的。

同时ES的date类型允许我们规定格式，可以使用的格式有：

yyyy-MM-dd HH:mm:ss

yyyy-MM-dd

epoch_millis（毫秒值）

1.规定格式如下：|| 表示或者

PUT my_index
{
  "mappings": {
      "properties": {
        "date": {
          "type":   "date",
          "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
        }
      }
  }
}

注意：一旦我们规定了格式，如果新增数据不符合这个格式，ES将会报错mapper_parsing_exception。
```

### 本篇内容到此结束  下一篇我们继续

