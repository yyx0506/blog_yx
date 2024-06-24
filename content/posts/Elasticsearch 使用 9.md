
---
title: "Elasticsearch metrics aggs 聚合"
date: 2022-05-23
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

### Elasticsearch 使用 9



本篇博客 我们讲述 ES 的 metrics aggs  如下图所示

![](/img/es9infos1.png)

```
通过上图我们可以看到，一共有18种类型 metrics agg （7.7.1 之前） 但是es 的更新很快毕竟 用的人太多了
所以，对于每一种聚合类型，我们都去详细学习并掌握是比较费时间的，个人建议可以按如下方式学习：
1）了解每种聚合类型的使用场景，简单而言，就是知道每种聚合是干嘛的，能对数据做怎样的分析；
2）掌握常用的聚合操作，了解其注意事项和重要参数；
```

```
Metrics Aggregations 指标分析类型，就是一些数学运算，对文档字段进行统计分析，类似于 sql 的 COUNT() 、 SUM() 、 MAX() 等统计方法。
```



#### 1: 四个基本统计聚合 如下图所示

![](/img/es9infos2.png)

```
这四个函数在很多数据库已经语言中都有 avg max min sum
我们以一个简单案例来举例 获取全部用户的年龄总数
GET test/_search?size=0
{
  "aggs": {
    "min_age": {
      "sum": {
        "field": "age"
      }
    }
  }
}
```



#### 2:Value Count value计数聚合如下图所示

![](/img/es9infos3.png)

```
GET test/_search?size=0
{
  "aggs": {
    "types_count": {
      "value_count": {
        "field": "name.keyword"
      }
    }
  }
}
```



#### 3:Stats 统计聚合如下图所示

![](/img/es9infos4.png)

```
就是一个聚合函数，包含了上述5种聚合，简单看个示例，学习语法：
GET test/_search?size=0
{
  "aggs": {
    "ages_stats": {
      "stats": {
        "field": "age"
      }
    }
  }
}
结果会把所有信息都运算一遍
"aggregations" : {
    "ages_stats" : {
      "count" : 6,
      "min" : 21.0,
      "max" : 269.0,
      "avg" : 67.83333333333333,
      "sum" : 407.0
    }
  }
例如可以先进bucket aggs 聚合后在 进行metrics aggs
GET test/_search?size=0
{
  "aggs": {
    "age_25": {
      "range": {
        "field": "age",
        "ranges": [
          {
            "from": 20,
            "to": 25
          }
        ]
      },
      "aggs": {
    "ages_stats": {
      "stats": {
        "field": "age"
      }
    }
  }
    }
  }
  
}

```



#### 4:Weighted Avg 加权平均聚合 如下图所示

![](/img/es9infos5.png)

```
加权平均聚合和Avg Agg类似，掌握其计算公式： ∑(value * weight) / ∑(weight)。
例如 可以 计算一些我们想生成的指数类型之类
GET test1/_search?size=0
{
  "query": {"match": {
    "name.keyword": "aaa"
  }}, 
  "aggs": {
    "weighted_grade": {
      "weighted_avg":{
        "value":{
          "field":"high"
        },
        "weight":{
          "field":"weight"
        }
      }
    }
  }
}

GET test1/_search?size=0
{
  "aggs": {
    "weighted_grade": {
      "weighted_avg": {
        "value":{
          "script":"doc.grade.value + 1"
        },
        "weight":{
          "field":"weight"
        }
      }
    }
  }
}
这个功能需要探索 目前了解的是 可以通过 脚本注入的方式将 初始之改变 grade 可以是一个多个数字的列表 weight 只能是一个 值
如果聚合遇到一个文档有多个权值(例如，权值字段是一个多值字段)，它将抛出一个异常。如果您遇到这种情况，您将需要为weight字段指定一个脚本，并使用该脚本将多个值组合成一个要使用的值，此单一权重将独立应用于从value字段中提取的每个值，值和权重都可以从脚本而不是字段派生。


```



#### 5:cardinality 基数聚合如下图所示

![](/img/es9infos6.png)

```
场景实例 统计一共有多少个不同的名字 类似于去重
GET test/_search?size=0
{
  "aggs": {
    "distinct_name": {
      "cardinality": {
        "field": "name.keyword"
      }
    }
  }
}

```

#### 6:Top Hits 热门匹配聚合如下图所示

![](/img/es9infos7.png)

```
场景实例获取不同名字的拥堵最大年龄的用户 前三位 的名字 和tag
GET test/_search?size=0
{
  "aggs": {
    "top_name": {
      "top_hits": {
        "sort": [{
          "age": {
            "order": "desc"
          }
        }], 
        "_source": {"includes": ["name.keyword","tags.keyword"]}, 
        "size": 3
      }
    }
  }
}

```

#### 7:Top Metrics 最高度量标准聚合如下图所示

![](/img/es9infos8.png)

```
GET test/_search?size=0
{
  "aggs": {
    "distinct_name": {
      "top_metrics": {
        "field": "name.keyword"
      }
    }
  }
}
```

#### 8: 剩余的8种聚合 如下图所示

![](/img/es9infos9.png)