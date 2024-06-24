
---
title: "Elasticsearch Bucket aggregations"
date: 2022-05-21
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


### Elasticsearch 使用 7

```
这篇文章主要是讲述Elasticsearch 的聚合分类 如下图所示
```

![](/img/es7infos1.png)

```
目前主流的还是使用  Bucket aggregations 分桶聚合  Metric aggregations 指标聚合 Pipeline  aggregations 管道聚合
tips ： aggregations关键字可使用aggs代替
```

#### 1: 分桶聚合 Bucket aggregations

```
使用场景分析 ： 我们在jd tm tb 买东西时最相要看到的就是 销量 评价 分桶 就是可以将相同内容的 东西进行分桶。
比如 小米手机 华为手机 苹果手机 和其他手机 进行分桶 对应出来的就是5个桶 小米手机下可以分为红米 小米 又是两个桶
每个型号又可以分为不同的桶。
所以即按照一定的规则将文档分配到不同的桶中，达到分类分析的目的就是分桶聚合 ，目前es 已经有25种 分桶聚合的类型了
```

##### 如下图所示

![](/img/es7infos2.png)

```
聚合分析是数据库中重要的功能特性，完成对一个查询的数据集中数据的聚合计算，如：找出某字段（或计算表达式
的结果）的最大值、最小值，计算和、平均值等。Elasticsearch作为搜索引擎兼数据库，同样提供了强大的聚合分析
能力。
对一个数据集求最大、最小、和、平均值等指标的聚合，在ES中称为指标聚合 metric 而关系型数据库中除了有聚合
函数外，还可以对查询出的数据进行分组group by，再在组上进行指标聚合。在 ES 中group by 称为分桶，桶聚合
bucketing
Elasticsearch聚合分析语法 在查询请求体中以aggregations节点按如下语法定义聚合分析：

"aggregations" : { 
		"<aggregation_name>" : { <!--聚合的名字 --> 
				"<aggregation_type>" : { <!--聚合的类型 -->
        		<aggregation_body> <!--聚合体：对哪些字段进行聚合 --> }
        		[,"meta" : { [<meta_data_body>] } ]? <!--元 --> [,"aggregations" : { [<sub_aggregation>]+ } 
        		]? <!--在聚合里面在定义子聚合 --> 
				}
}
```



#### 1: terms 术语聚合 如下图所示

![](/img/es7infos3.png)

```
其中 ES 检索时会把所有的对象返回 如果不希望返回 就指定 "size": 0 对于text类型的文本聚合时要指定keyword
aggs 是aggregations 的简写 same_name 是我们自己取的聚合后的名字 terms 是我们指定的聚合类型 fild 是聚合文本
GET test/_search
{
  "size": 0,
  "aggs": {
    "same_name": {
      "terms": {
        "field": "name.keyword"
      }
    }
  }
}
```

#### 2:Rare Terms  稀有术语聚合如下图所示

![](/img/es7infos4.png)

```
在 Terms Aggs 中，聚合结果的排序是默认根据 doc_count 的值降序排列，但在实际使用过程中，我们有时候希望根据 doc_count 的值升序排列，这个时候就应该使用  Rare Terms【之所以不使用 Terms aggs再去改变排序规则，是因为聚合精度问题】
注意max_doc_count参数：术语出现的最大文档数【返回的bucket 的 doc_count <= 该值】，默认值为1，最大值为100。

GET test/_search
{
  "size": 0,
  "aggs": {
    "same_name": {
      "rare_terms": {
        "field": "name.keyword",
        "max_doc_count": 10
      }
    }
  }
}
```

#### 3:Histogram 直方图聚合如下图所示

![](/img/es7infos5.png)

```
例如 我们按照 用户的年龄进行聚合 可以得到用户在各年龄端的分布 interval 代表的是区间分布
GET test/_search
{
  "size": 0,
  "aggs": {
    "same_name": {
      "histogram": {
        "field": "age",
        "interval": 10 
      }
    }
  }
}
```

#### 4:Date histogram 日期直方图聚合如下图所示

![](/img/es7infos6.png)

```
GET test/_search
{
  "size": 0,
  "aggs": {
    "price": {
      "date_histogram": {
        "field": "datatime",
        "calendar_interval": "day",
        "format": "yyyy-MM-dd"
      }
    }
  }
}
```

#### 5:Auto-interval Date Histogram 自动间隔日期直方图聚合如下图所示

![](/img/es7infos7.png)

```
场景示例：还是通过博客的创建时间做聚合，这次我希望返回3个bucket
GET  test/_search
{
  "size": 0,
  "aggs": {
    "createTime": {
      "auto_date_histogram": {
        "field": "datatime",
        "format": "yyyy-MM-dd",
        "buckets": 3
      }
    }
  }
}
```

#### 6:Range 范围聚合如下图所示

![](/img/es7infos8.png)

```
场景示例：查看年龄在20-25 25-30 以及30以上的用户数量
GET test/_search
{
  "size": 0,
  "aggs": {
    "user_ages": {
      "range": {
        "field": "age",
        "ranges": [
          {
            "from": 20,
            "to": 25
          },
          {
            "from": 25,
            "to": 30
          },
          {
            "from": 30
          }
        ]
      }
    }
  }
}

```

#### 7:Date Range 日期范围聚合如下图所示

![](/img/es7infos9.png)

```
场景示例：获取过去到10个月之前的所有商品总数和10个月之前的商品总数：
POST /product/_search
{
  "aggs": {
    "range": {
      "date_range": {
        "field": "date",
        "format": "yyyy-MM",
        "ranges": [
          {
            "to": "now-10M/M"
          },
          {
            "from": "now-10M/M"
          }
        ]
      }
    }
  }
}

注意：to：date < 现在减去10个月，向下舍入到月初；from：date > = 现在减去10个月，向下舍入到月初。
```

#### 8:IP Range IP范围聚合如下图所示

![](/img/es7infos10.png)

```
POST /ip_addresses/_search
{
  "size": 10,
  "aggs": {
    "ip_ranges": {
      "ip_range": {
        "field": "ip",
        "ranges": [
          {
            "to": "10.0.0.5"
          },
          {
            "from": "10.0.0.1"
          }
        ]
      }
    }
  }
}
```

#### 9:Composite 复合聚合如下图所示

![](/img/es7infos11.png)

```
1）对于Composite Agg 需要看两个示例，一个是翻页的示例：
POST /teamwork_task/_search
{
  "size": 0,
  "aggs": {
    "my_buckets": {
      "composite": {
        "size": 2,
        "sources": [
          {
            "customName": {
              "terms": {
                "field": "team_id"
              }
            }
          }
        ]
      }
    }
  }
}
结果：
  "aggregations": {
    "my_buckets": {
      "after_key": {
        "customName": 135
      },
      "buckets": [
        {
          "key": {
            "customName": 128
          },
          "doc_count": 2
        },
        {
          "key": {
            "customName": 135
          },
          "doc_count": 3
        }
      ]
    }
  }
}
翻页查询：查询 body不变，添加上次返回的 after即可
POST /teamwork_task/_search
{
  "size": 0,
  "aggs": {
    "my_buckets": {
      "composite": {
        "size": 2,
        "sources": [
          {
            "customName": {
              "terms": {
                "field": "team_id"
              }
            }
          }
        ],
        "after": {
          "customName": 135
        }
      }
    }
  }
}

2）问卷结果统计示例：假设有A、B两道题，每题都有2个答案，那么 Composite聚合就可以得到所有可能组合的答案的问卷数
POST /question_index/_search
{
  "size": 0,
  "aggs": {
    "my_buckets": {
      "composite": {
        "sources": [
          {
            "question_A": {
              "terms": {
                "field": "question_A"
              }
            }
          },
          {
            "question_B": {
              "terms": {
                "field": "question_B"
              }
            }
          }
        ]
      }
    }
  }
}
结果：
  "aggregations": {
    "my_buckets": {
      "after_key": {
        "question_A": "B",
        "question_B": 1
      },
      "buckets": [{
            "key": {
                "question_A": "a",
                "question_B": "c"
            },
            "doc_count": 1
        },
        {
            "key": {
                "question_A": "a",
                "question_B": "d"
            },
            "doc_count": 1
        },
        {
            "key": {
                "question_A": "b",
                "question_B": "c"
            },
            "doc_count": 1
        },
        {
            "key": {
                "question_A": "b",
                "question_B": "d"
            },
            "doc_count": 1
        }]
    }
  }
```

#### 10:Filter 过滤器聚合 如下图所示

![](/img/es7infos12.webp)

```
比如 我们获取 名字是 qqqq 的所有人的平均年龄 
POST test/_search
{
    "size": 0, 
    "aggs" : {
        "a_ages" : {
            "filter" : { "term": { "name.keyword": "qqqq" } },
            "aggs" : {
                "avg_price" : { "avg" : { "field" : "age" } }
            }
        }
    }
}
```



本篇博客暂时讲到这里 ，下一篇我们继续讲解es 的使用



