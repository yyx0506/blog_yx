
---
title: "Elasticsearch pipeline aggs  聚合"
date: 2022-05-24
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


### Elasticsearch 使用 10



这一篇我们讲一下 pipeline aggs 如图所示


![](/img/es10infos1.webp)


```
在掌握了Bucket Agg 和 Metric Agg 后，对于很多聚合分析场景，就已经可以得心应手了，如果有更为复杂的，比如说二次聚合，可能就需要用到管道聚合 Pipeline Agg 了。

什么叫 Pipeline Agg 呢？就是管道聚合：对其他聚合结果进行二次聚合。注意，管道聚合不能具有子聚合，但是根据其类型，它可以引用buckets_path 允许管道聚合链接的另一个管道。例如，您可以将两个导数链接在一起以计算第二个导数（即导数的导数）。

在系统学习管道聚合之前，我们需要先掌握管道聚合的必填参数 buckets_path 的语法。
```



#### 1:buckets_path 的语法

```
与操作的聚合对象同级，就是 Agg_Name
POST /_search
{
    "aggs": {
        "my_date_histo":{
            "date_histogram":{
                "field":"timestamp",
                "calendar_interval":"day"
            },
            "aggs":{
                "the_sum":{
                    "sum":{ "field": "lemmings" }
                },
                "the_movavg":{
                    "moving_avg":{ "buckets_path": "the_sum" }
                }
            }
        }
    }
}
简单来说就是一个 aggs 分桶完成后还是达不到想要的效果就需要二次聚合  核心就是 buckets_path  选择上次聚合后效果 

```

#### 2:与操作的聚合对象不同级，就是 Agg_Name的相对路径，多个 Agg_Name间使用“>”连接

```
POST /_search
{
    "aggs" : {
        "sales_per_month" : {
            "date_histogram" : {
                "field" : "date",
                "calendar_interval" : "month"
            },
            "aggs": {
                "sales": {
                    "sum": {
                        "field": "price"
                    }
                }
            }
        },
        "max_monthly_sales": {
            "max_bucket": {
                "buckets_path": "sales_per_month>sales"
            }
        }
    }
}
```

#### 3:操作对象是多值桶某个key 的聚合，就是 Agg_Name1['keyName']>Agg_Name2

```
POST /_search
{
    "aggs" : {
        "sales_per_month" : {
            "date_histogram" : {
                "field" : "date",
                "calendar_interval" : "month"
            },
            "aggs": {
                "sale_type": {
                    "terms": {
                        "field": "type"
                    },
                    "aggs": {
                        "sales": {
                            "sum": {
                                "field": "price"
                            }
                        }
                    }
                },
                "hat_vs_bag_ratio": {
                    "bucket_script": {
                        "buckets_path": {
                            "hats": "sale_type['hat']>sales",
                            "bags": "sale_type['bag']>sales"
                        },
                        "script": "params.hats / params.bags"
                    }
                }
            }
        }
    }
}
```

#### 4:特殊路径

```
POST /_search
{
    "aggs": {
        "my_date_histo": {
            "date_histogram": {
                "field":"timestamp",
                "calendar_interval":"day"
            },
            "aggs": {
                "the_movavg": {
                    "moving_avg": { "buckets_path": "_count" }
                }
            }
        }
    }
}

或者
POST /sales/_search
{
"size": 0,
"aggs": {
    "histo": {
    "date_histogram": {
        "field": "date",
        "calendar_interval": "day"
    },
    "aggs": {
        "categories": {
        "terms": {
            "field": "category"
        }
        },
        "min_bucket_selector": {
        "bucket_selector": {
            "buckets_path": {
            "count": "categories._bucket_count"
            },
            "script": {
            "source": "params.count != 0"
            }
        }
        }
    }
    }
}
}

了解了buckets_path 的语法，接下来我们就详细学习管道聚合，在学习之前，我们要知道管道聚合根据输出结果的位置分为Parent【结果内嵌到现有的聚合分析结果中】 和 Sibling【结果和现有分析结果同级】 两类。
```

#### 5:实现having 效果

```
POST /book/_search 
{"size": 0,
                "aggs":
                    {"group_by_price":
                         {"range": {"field": "price",
                                    "ranges": [{"from": 0, "to": 200},
                                               {"from": 200, "to": 400},
                                               {"from": 400, "to": 1000}]},
                          "aggs": {"average_price": 
                         				 {"avg": {"field": "price"}},
                                   "having": {
                                       "bucket_selector": {
                                           "buckets_path": {"avg_price": "average_price"},
                                           "script": {"source": "params.avg_price >= 200 "
}}}}}}}

```

#### 6: Bucket Script 桶脚本聚合 如下图所示

![](/img/es10infos2.webp)

```
场景示例：计算出每月T恤销售额与总销售额的比例百分比
POST /sales/_search
{
    "size": 0,
    "aggs" : {
        "sales_per_month" : {
            "date_histogram" : {
                "field" : "date",
                "calendar_interval" : "month"
            },
            "aggs": {
                "total_sales": {
                    "sum": {
                        "field": "price"
                    }
                },
                "t-shirts": {
                "filter": {
                    "term": {
                    "type": "t-shirt"
                    }
                },
                "aggs": {
                    "sales": {
                    "sum": {
                        "field": "price"
                    }
                    }
                }
                },
                "t-shirt-percentage": {
                    "bucket_script": {
                        "buckets_path": {
                        "tShirtSales": "t-shirts>sales",
                        "totalSales": "total_sales"
                        },
                        "script": "params.tShirtSales / params.totalSales * 100"
                    }
                }
            }
        }
    }
}
```



#### 7:Bucket Selector 桶选择器聚合如下图所示

![](/img/es10infos1.webp)

```
场景示例：获取当月总销售额超过200的存储分区桶
POST /sales/_search
{
    "size": 0,
    "aggs" : {
        "sales_per_month" : {
            "date_histogram" : {
                "field" : "date",
                "calendar_interval" : "month"
            },
            "aggs": {
                "total_sales": {
                    "sum": {
                        "field": "price"
                    }
                },
                "sales_bucket_filter": {
                    "bucket_selector": {
                        "buckets_path": {
                        "totalSales": "total_sales"
                        },
                        "script": "params.totalSales > 200"
                    }
                }
            }
        }
    }
}
```



#### 8: Bucket Sort 桶排序聚合如下图所示

![](/img/es10infos4.png)

```
场景示例：按降序返回总销售额最高的3个月相对应的存储桶
POST /sales/_search
{
    "size": 0,
    "aggs" : {
        "sales_per_month" : {
            "date_histogram" : {
                "field" : "date",
                "calendar_interval" : "month"
            },
            "aggs": {
                "total_sales": {
                    "sum": {
                        "field": "price"
                    }
                },
                "sales_bucket_sort": {
                    "bucket_sort": {
                        "sort": [
                        {"total_sales": {"order": "desc"}}
                        ],
                        "size": 3，				
                "from": 0
                    }
                }
            }
        }
    }
}
```



#### 9:Cumulative Cardinality  累积基数聚合如下图所示

![](/img/es10infos5.png)

```
场景示例1：获取网站每天的新访问者总的累计数量【后一天会累加前一天的，就是以第一天为基准】
GET /user_hits/_search
{
    "size": 0,
    "aggs" : {
        "users_per_day" : {
            "date_histogram" : {
                "field" : "timestamp",
                "calendar_interval" : "day"
            },
            "aggs": {
                "distinct_users": {
                    "cardinality": {
                        "field": "user_id"
                    }
                },
                "total_new_users": {
                    "cumulative_cardinality": {
                        "buckets_path": "distinct_users"
                    }
                }
            }
        }
    }
}
结果：
"aggregations": {
    "users_per_day": {
        "buckets": [
            {
            "key_as_string": "2019-01-01T00:00:00.000Z",
            "key": 1546300800000,
            "doc_count": 2,
            "distinct_users": {
                "value": 2
            },
            "total_new_users": {
                "value": 2
            }
            },
            {
            "key_as_string": "2019-01-02T00:00:00.000Z",
            "key": 1546387200000,
            "doc_count": 2,
            "distinct_users": {
                "value": 2
            },
            "total_new_users": {
                "value": 3
            }
            },
            {
            "key_as_string": "2019-01-03T00:00:00.000Z",
            "key": 1546473600000,
            "doc_count": 3,
            "distinct_users": {
                "value": 3
            },
            "total_new_users": {
                "value": 4
            }
            }
        ]
    }
}
}

结果分析：
请注意，第二天2019-01-02拥有两个不同的用户，
但total_new_users累积管道agg生成的指标仅增加到三个。
这意味着当天的两个用户中只有一个是新用户，
而在前一天已经看到了另一个用户。
第三天再次发生这种情况，三个用户中只有一个是全新的。

添加聚合：derivative 增量累积基数
场景示例2：获取网站每天增加了多少新用户【数据以前一天为基准】
GET /user_hits/_search
{
    "size": 0,
    "aggs" : {
        "users_per_day" : {
            "date_histogram" : {
                "field" : "timestamp",
                "calendar_interval" : "day"
            },
            "aggs": {
                "distinct_users": {
                    "cardinality": {
                        "field": "user_id"
                    }
                },
                "total_new_users": {
                    "cumulative_cardinality": {
                        "buckets_path": "distinct_users"
                    }
                },
                "incremental_new_users": {
                    "derivative": {
                        "buckets_path": "total_new_users"
                    }
                }
            }
        }
    }
}
结果：
"aggregations": {
    "users_per_day": {
        "buckets": [
            {
            "key_as_string": "2019-01-01T00:00:00.000Z",
            "key": 1546300800000,
            "doc_count": 2,
            "distinct_users": {
                "value": 2
            },
            "total_new_users": {
                "value": 2
            }
            },
            {
            "key_as_string": "2019-01-02T00:00:00.000Z",
            "key": 1546387200000,
            "doc_count": 2,
            "distinct_users": {
                "value": 2
            },
            "total_new_users": {
                "value": 3
            },
            "incremental_new_users": {
                "value": 1.0
            }
            },
            {
            "key_as_string": "2019-01-03T00:00:00.000Z",
            "key": 1546473600000,
            "doc_count": 3,
            "distinct_users": {
                "value": 3
            },
            "total_new_users": {
                "value": 4
            },
            "incremental_new_users": {
                "value": 1.0
            }
            }
        ]
    }
}
}
```



#### 10 :Cumulative Sum 累积总和聚合 如下图所示
![](/img/es10infos6.webp)


```
场景示例：计算到当月为止，每月累计销售金额的总和
POST /sales/_search
{
    "size": 0,
    "aggs" : {
        "sales_per_month" : {
            "date_histogram" : {
                "field" : "date",
                "calendar_interval" : "month"
            },
            "aggs": {
                "sales": {
                    "sum": {
                        "field": "price"
                    }
                },
                "cumulative_sales": {
                    "cumulative_sum": {
                        "buckets_path": "sales"
                    }
                }
            }
        }
    }
}
```



#### 11:4个Parent类型的管道聚合 如图所示

![](/img/es10infos7.webp)



#### 12:六种统计类管道聚合 如图所示

![](/img/es10infos8.png)

```
和指标聚合类似，Avg、Max、Min、Sum，一看就知道是什么意思，简单看一个示例即可。

场景示例：计算每月销售额总量的平均值
POST /_search
{
"size": 0,
"aggs": {
    "sales_per_month": {
    "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
    },
    "aggs": {
        "sales": {
        "sum": {
            "field": "price"
        }
        }
    }
    },
    "avg_monthly_sales": {
    "avg_bucket": {
        "buckets_path": "sales_per_month>sales"
    }
    }
}
}
结果：
{
"took": 11,
"timed_out": false,
"_shards": ...,
"hits": ...,
"aggregations": {
    "sales_per_month": {
        "buckets": [
            {
            "key_as_string": "2015/01/01 00:00:00",
            "key": 1420070400000,
            "doc_count": 3,
            "sales": {
                "value": 550.0
            }
            },
            {
            "key_as_string": "2015/02/01 00:00:00",
            "key": 1422748800000,
            "doc_count": 2,
            "sales": {
                "value": 60.0
            }
            },
            {
            "key_as_string": "2015/03/01 00:00:00",
            "key": 1425168000000,
            "doc_count": 2,
            "sales": {
                "value": 375.0
            }
            }
        ]
    },
    "avg_monthly_sales": {
        "value": 328.33333333333333
    }
}
}

对于 Stats Bucket 统计数据桶聚合，我们看下响应结果，知道统计了哪些指标即可：
"aggregations": {
    "sales_per_month": {
        "buckets": [
            {
            "key_as_string": "2015/01/01 00:00:00",
            "key": 1420070400000,
            "doc_count": 3,
            "sales": {
                "value": 550.0
            }
            },
            {
            "key_as_string": "2015/02/01 00:00:00",
            "key": 1422748800000,
            "doc_count": 2,
            "sales": {
                "value": 60.0
            }
            },
            {
            "key_as_string": "2015/03/01 00:00:00",
            "key": 1425168000000,
            "doc_count": 2,
            "sales": {
                "value": 375.0
            }
            }
        ]
    },
    "stats_monthly_sales": {
        "count": 3,
        "min": 60.0,
        "max": 550.0,
        "avg": 328.3333333333333,
        "sum": 985.0
    }
}
}

Extended Stats Bucket 扩展统计数据桶聚合，也直接看统计结果：
示例结果：
    "stats_monthly_sales": {
        "count": 3,
        "min": 60.0,
        "max": 550.0,
        "avg": 328.3333333333333,
        "sum": 985.0,
        "sum_of_squares": 446725.0,
        "variance": 41105.55555555556,
        "std_deviation": 202.74505063146563,
        "std_deviation_bounds": {
        "upper": 733.8234345962646,
        "lower": -77.15676792959795
        }
    }
}
```



#### 13:Percentiles Bucket 百分数桶聚合 如图所示

![](/img/es10infos9.png)

```
场景示例：计算每月总销售额存储桶对应的百分比位置的金额
POST /sales/_search
{
    "size": 0,
    "aggs" : {
        "sales_per_month" : {
            "date_histogram" : {
                "field" : "date",
                "calendar_interval" : "month"
            },
            "aggs": {
                "sales": {
                    "sum": {
                        "field": "price"
                    }
                }
            }
        },
        "percentiles_monthly_sales": {
            "percentiles_bucket": {
                "buckets_path": "sales_per_month>sales",
                "percents": [ 25.0, 50.0, 75.0 ]
            }
        }
    }
}
结果：
"aggregations": {
    "sales_per_month": {
        "buckets": [
            {
            "key_as_string": "2015/01/01 00:00:00",
            "key": 1420070400000,
            "doc_count": 3,
            "sales": {
                "value": 550.0
            }
            },
            {
            "key_as_string": "2015/02/01 00:00:00",
            "key": 1422748800000,
            "doc_count": 2,
            "sales": {
                "value": 60.0
            }
            },
            {
            "key_as_string": "2015/03/01 00:00:00",
            "key": 1425168000000,
            "doc_count": 2,
            "sales": {
                "value": 375.0
            }
            }
        ]
    },
    "percentiles_monthly_sales": {
        "values" : {
            "25.0": 375.0,
            "50.0": 375.0,
            "75.0": 550.0
        }
    }
}
}
```

