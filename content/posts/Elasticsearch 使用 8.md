
---
title: "Elasticsearch Filters 聚合"
date: 2022-05-22
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


### Elasticsearch 使用 8



```
上一篇文章我们讲述了 bucket aggs 的10种聚合方式 ，这次我们继续讲其他的聚合方式
```



#### 11:Filters 过滤器集合聚合 如下图所示

![](/img/es8infos1.png)

```
查询name 是 qqqq 和 ass 的个数
GET test/_search
{
  "size":0,
  "aggs": {
    "messages": {
      "filters": {
        "filters": {
          "messages":{
            "terms": {
              "name": [
                "qqqq",
                "ass"
              ]
            }
          }
          }
        }
      }
    }
}
```

#### 12:Global 全局聚合如下图所示

![](/img/es8infos1.png)

```
GET test/_search
{
  "query": {"match": {
    "name.keyword": "qqqq"
  }}, 
  "size": 0,
  "aggs": {
    "all_user": {
      "global": {}, 
      "aggs": {
        "avg_age": {
          "avg": {
            "field": "age"
          }
        }
      }
    },
    "age":{
      "avg": {
        "field": "age"
      }
    }
  }
}

```



#### 13:Missing 缺少聚合如下图所示

![](/img/es8infos3.webp)

```
场景示例：获取没有标价的商品的总数
POST test/_search?size=0
{
  "aggs": {
    "products_without_a_age": {
      "missing": {
        "field": "age"
      }
    }
  }
}
```



#### 14: Adjacency Matrix 邻接矩阵聚合如下图所示

![](/img/es8infos4.png)
![](/img/es8infos5.png)

#### 15: 3种地理位置的bucket聚合如下图所示
![](/img/es8infos6.webp)


#### 16:其余8种bucket聚合不是很常见如果遇到在详细分析 如下图所示

![](/img/es8infos7.png)