# ES 聚合

参见官方文档：https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html

聚合用于统计ES中的文档。

聚合语句的格式：

```json
"aggregations" : {
    "<aggregation_name>" : {
        "<aggregation_type>" : {
            <aggregation_body>
        }
        [,"meta" : {  [<meta_data_body>] } ]?
        [,"aggregations" : { [<sub_aggregation>]+ } ]?
    }
    [,"<aggregation_name_2>" : { ... } ]*
}
```



## 聚合类型

**桶（Bucket）聚合**

**指标（Metric）聚合**

**矩阵（Matrix）聚合** 实验特性

**管道（Pipeline）聚合** Pipeline Aggregations 是一组工作在其他聚合计算结果而不是文档集合的聚合。

其中，只有矩阵聚合不能使用脚本。



## 度量聚合

### 求平均数

```json
POST /follower/_search
{
  "aggs": {
    "avg_rename_days": {
      "avg": {
        "field": "gender"
      }
    }
  },
  "size": 0
}
```

这里把 size 设置为0的意思是只显示聚合数据。返回：

```json
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 10000,
      "relation" : "gte"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "avg_rename_days" : {
      "value" : 0.3915204490434261
    }
  }
}
```

对结果进行操作：

```json
POST /follower/_search
{
  "aggs": {
    "avg_rename_days": {
      "avg": {
        "field": "gender",
        "script" : {
              "lang": "painless",
              "source": "_value + params.correction",
              "params" : {
                  "correction" : 1
              }
          }
      }
    }
  },
  "size": 0
}
```

结果：

```json
{
  "took" : 6,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 10000,
      "relation" : "gte"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "avg_rename_days" : {
      "value" : 1.391520449043426
    }
  }
}
```



### 基数聚合

就是统计某个字段有多少种类型。在非数字类型字段进行统计前要执行：

```json
PUT /follower/_mapping
{
  "properties": {
    "v": { 
      "type":     "text",
      "fielddata": true
    }
  }
}
```

然后就可以统计了：

```json
POST /follower/_search
{
  "aggs": {
    "type_count": {
      "cardinality": {
        "field": "v",
        "precision_threshold": 1
      }
    }
  },
  "size": 0
}
```

结果：

```json
{
  "took" : 503,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 10000,
      "relation" : "gte"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "type_count" : {
      "value" : 1362
    }
  }
}
```

可以看到，v 字段的值有 1362 种。



### grades_stats

一口气把 avg ，sum，max，min 等等全部求出来：

```json
POST /follower/_search
{
  "aggs": {
    "stats": {
      "extended_stats": {
        "field": "gender"
      }
    }
  },
  "size": 0
}
```

结果：

```json
{
  "took" : 16,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 10000,
      "relation" : "gte"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "stats" : {
      "count" : 46677,
      "min" : -1.0,
      "max" : 1.0,
      "avg" : 0.3915204490434261,
      "sum" : 18275.0,
      "sum_of_squares" : 36441.0,
      "variance" : 0.6274174388613534,
      "std_deviation" : 0.7920968620448848,
      "std_deviation_bounds" : {
        "upper" : 1.9757141731331958,
        "lower" : -1.1926732750463436
      }
    }
  }
}
```



### max

max 很简单，就是找最大数。

```json
POST /sales/_search?size=0
{
    "aggs" : {
        "max_price" : { "max" : { "field" : "price" } }
    }
}
```

### min

min 就是找最小的数：

```json
POST /sales/_search?size=0
{
    "aggs" : {
        "min_price" : { "min" : { "field" : "price" } }
    }
}
```

### percentiles

percentile 百分位统计是用于评估当前数值分布情况的一种统计方法，比如50 percentile 是 98 ， 是指 50%的数值都在98以内，那么通过这里你就可以了解你数据的一个分布特征。常见的一个应用是我们制定 SLA 的时候常说 99% 的请求延迟都在100ms 以内，这个时候你就可以用 99 percentile 来查一下，看一下 99 percenttile 的值如果在 100ms 以内，那么恭喜你 SLA 达标了，否则就说明有问题。

```json
GET latency/_search
{
    "size": 0,
    "aggs" : {
        "load_time_outlier" : {
            "percentiles" : {
                "field" : "load_time" 
            }
        }
    }
}
```

默认是 [ 1, 5, 25, 50, 75, 95, 99 ]：

```json
{
    ...

   "aggregations": {
      "load_time_outlier": {
         "values" : {
            "1.0": 5.0,
            "5.0": 25.0,
            "25.0": 165.0,
            "50.0": 445.0,
            "75.0": 725.0,
            "95.0": 945.0,
            "99.0": 985.0
         }
      }
   }
}
```

可以指定百分数：

```json
GET latency/_search
{
    "size": 0,
    "aggs" : {
        "load_time_outlier" : {
            "percentiles" : {
                "field" : "load_time",
                "percents" : [95, 99, 99.9] 
            }
        }
    }
}
```

### percentile_ranks

理解了 percentile，那么 percentile rank 其实就是反向查询，比如我想看一下 100 在当前数值中处于哪一个范围内，你查一下它的 rank，发现是95，那么说明有95%的数值都在100以内。

```json
GET latency/_search
{
    "size": 0,
    "aggs" : {
        "load_time_ranks" : {
            "percentile_ranks" : {
                "field" : "load_time", 
                "values" : [500, 600]
            }
        }
    }
}
```

结果：

```json
{
    ...

   "aggregations": {
      "load_time_ranks": {
         "values" : {
            "500.0": 90.01,
            "600.0": 100.0
         }
      }
   }
}
```



### scripted_metric

写脚本来聚合：

```json
POST ledger/_search?size=0
{
    "query" : {
        "match_all" : {}
    },
    "aggs": {
        "profit": {
            "scripted_metric": {
                "init_script" : "state.transactions = []", 
                "map_script" : "state.transactions.add(doc.type.value == 'sale' ? doc.amount.value : -1 * doc.amount.value)",
                "combine_script" : "double profit = 0; for (t in state.transactions) { profit += t } return profit",
                "reduce_script" : "double profit = 0; for (a in states) { profit += a } return profit"
            }
        }
    }
}
```

### stats

 `min`, `max`, `sum`, `count` and `avg` 的统称：

```json
POST /exams/_search?size=0
{
    "aggs" : {
        "grades_stats" : { "stats" : { "field" : "grade" } }
    }
}
```

结果：

```json
{
    ...

    "aggregations": {
        "grades_stats": {
            "count": 2,
            "min": 50.0,
            "max": 100.0,
            "avg": 75.0,
            "sum": 150.0
        }
    }
}
```

### sum

计算和

```json
POST /sales/_search?size=0
{
    "query" : {
        "constant_score" : {
            "filter" : {
                "match" : { "type" : "hat" }
            }
        }
    },
    "aggs" : {
        "hat_prices" : { "sum" : { "field" : "price" } }
    }
}
```

结果：

```json
{
    ...
    "aggregations": {
        "hat_prices": {
           "value": 450.0
        }
    }
}
```

### value_count

有多少种值：

```json
POST /sales/_search?size=0
{
    "aggs" : {
        "types_count" : { "value_count" : { "field" : "type" } }
    }
}
```

结果：

```json
{
    ...
    "aggregations": {
        "types_count": {
            "value": 7
        }
    }
}
```



### median_absolute_deviation

绝对中位差是统计学中的一个术语。

绝对中位差是一种统计离差的测量。而且，MAD是一种鲁棒统计量，比标准差更能适应数据集中的异常值。对于标准差，使用的是数据到均值的距离平方，所以大的偏差权重更大，异常值对结果也会产生重要影响。对于MAD，少量的异常值不会影响最终的结果。

由于MAD是一个比样本方差或者标准差更鲁棒的度量，它对于不存在均值或者方差的分布效果更好，比如柯西分布。

例子：

```json
GET reviews/_search
{
  "size": 0,
  "aggs": {
    "review_average": {
      "avg": {
        "field": "rating"
      }
    },
    "review_variability": {
      "median_absolute_deviation": {
        "field": "rating" 
      }
    }
  }
}
```

结果：

```json
{
  ...
  "aggregations": {
    "review_average": {
      "value": 3.0
    },
    "review_variability": {
      "value": 2.0
    }
  }
}
```



## 桶聚合

### adjacency_matrix 

分组统计文档数量：

```json
PUT /emails/_bulk?refresh
{ "index" : { "_id" : 1 } }
{ "accounts" : ["hillary", "sidney"]}
{ "index" : { "_id" : 2 } }
{ "accounts" : ["hillary", "donald"]}
{ "index" : { "_id" : 3 } }
{ "accounts" : ["vladimir", "donald"]}

GET emails/_search
{
  "size": 0,
  "aggs" : {
    "interactions" : {
      "adjacency_matrix" : {
        "filters" : {
          "grpA" : { "terms" : { "accounts" : ["hillary", "sidney"] }},
          "grpB" : { "terms" : { "accounts" : ["donald", "mitt"] }},
          "grpC" : { "terms" : { "accounts" : ["vladimir", "nigel"] }}
        }
      }
    }
  }
}
```

结果：

```json
{
  "took": 9,
  "timed_out": false,
  "_shards": ...,
  "hits": ...,
  "aggregations": {
    "interactions": {
      "buckets": [
        {
          "key":"grpA",
          "doc_count": 2
        },
        {
          "key":"grpA&grpB",
          "doc_count": 1
        },
        {
          "key":"grpB",
          "doc_count": 2
        },
        {
          "key":"grpB&grpC",
          "doc_count": 1
        },
        {
          "key":"grpC",
          "doc_count": 1
        }
      ]
    }
  }
}
```

