
---
title: "Elasticsearch 排序 查询 匹配"
date: 2022-05-20
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

### Elasticsearch 使用 6

```
我们都知到所有的数据库最复杂也最难的就是查询了，接下来我们这一篇文章主要讲的就是查询
首先我们先创建几条数据
下面第一个是kibana 的写法 第二个是python 的 index 的写法  第三个是 update 的写法 二者都可以更新或者新增，update需要将 body再套一层 doc
PUT /test/_doc/4 
{
   "name": "das",
    "age": 21,
    "desc": "操作还可以",
    "tags": ["够用", "还行", "凑合"]
}
es = await AioESService.get()
body = {
      "name": "qqqq",
      "age": 30,
      "desc": "操作猛如虎",
      "tags": ["强", "还行", "大神"]
}
await es.index(index='test', id = 3, body=body)
es = await AioESService.get()
body = {
      "name": "asx",
      "age": 36,
      "desc": "没有操作",
      "tags": ["菜", "太菜", "玩不了"]
}
await es.update(index='test', id = 5, body={"doc": body})

```

##### 现在有六条数据 如下图显示

![](/img/es6infos1.png)



##### 下面我们来做一下搜索看看

```
GET test/_search
{
  "query" : {
    "match":{
      "name":"das"
    }
  }
}
es = await AioESService.get()
body = {
   "query":{
         "match":{
            "name":"ass"
									}
						}
}
result = await es.search(index='test',body=body)
```

##### 可以看到如下图所示


![](/img/es6infos2.png)




```
其中hits ： 索引和文档的信息 查询的所有结果 score 是匹配度 从上往下排列
```

```
当我们只要某些数据时 可以在查询的时候通过 “_source” 去指定
GET test/_search
{
  "query" : {
    "match":{
      "name":"das"
    }
  }
  , "_source": ["name","desc"]
}
```



##### 排序以及分页

```
GET test/_search
{
  "query":{
    "match":{
      "name":"qqqq"
            }
          },
    "sort": [{
      "age":{
        "order": "desc"
      }
    }]
}
es = await AioESService.get()
body = {
     "query":{
       "match":{
       "name":"qqqq"
        }
     },
      "sort": [{
        "age": {
           "order": "desc"
          }
        }]
}
await es.search(index='test',body=body)
```

##### 可以看到结果如下图所示
![](/img/es6infos3.png)


```
可以在加入 from 和 size参数 进行分页 其中size 控制为一页多少个， from 代表从哪个地方开始 
GET test/_search
{
  "query":{
    "match":{
      "name":"qqqq"
            }
          },
    "sort": [{
      "age":{
        "order": "desc"
      }
    }],
    "from": 0,
    "size": 1
}
```



##### 布尔值查询 多条件查询

```
must 相当于 mysql and
GET test/_search
{
  "query":{
    "bool":{
      "must":[
        {
          "match": {
            "name": "qqqq"
          }
        },
        {
          "match":{
            "age":30
          }
        }]
            }
          }
}
should 相当于 mysql or
GET test/_search
{
  "query":{
    "bool":{
      "should":[
        {
          "match": {
            "name": "das"
          }
        },
        {
          "match":{
            "age":24
          }
        }]
            }
          }
}
must_not 相当于 not 
GET test/_search
{
  "query":{
    "bool":{
      "must_not":[
        {
          "match": {
            "name": "das"
          }
        },
        {
          "match":{
            "age":24
          }
        }]
            }
          }
}
```

#### 制定过滤器 filter

```
filter 是和 must 同级的
GET test/_search
{
  "query":{
    "bool":{
      "must":[
        {
          "match": {
            "name": "qqqq"
          }
        }
        ],
        "filter": [
          {"range": {
            "age": {
              "gte": 28
            }
          }}
]
}
}
}
```

##### 匹配多个条件

```
多个条件使用空格 隔开即可 而且会自动分词
GET test/_search
{
  "query":{
    "match": {
      "tags": "还行 大神"
    }
  }
}
```

##### 精确查询！

```
term 查询 是直接 通过倒排索引指定的词条进程精确查找的！
关于分词：
term , 直接查询精确的
match ,会使用分词器解析 （先分析文档，然后通过分析的文档进行查询）
```

