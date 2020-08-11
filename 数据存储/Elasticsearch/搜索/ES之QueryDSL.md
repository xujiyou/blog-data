# ES 之 Query DSL

ES 入门之后，就要来学习一下 Query DSL 了，因为 ES 中的全部搜索功能全局集中在 Query DSL 中了，学会 Query DSL 就可以随意搜索你自己的文档了。不过这块内容还是比较多的，需要花一段时间。

本文完全参考官方文档：[Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)

## 搜索前提

默认地，ES的搜索结果是根据相关性得分排序的，相关性得分是一个文档根据匹配得到的分数。

query 子句与 filter 子句

query 子句的意思是：此文档与此查询子句的匹配程度如何？query 子句是要计算文档得分的。

filter 子句的意思是：此文档是否与此查询子句匹配？filter 子句不需要就算文档得分。

bool 子句中的 must_not 也是一种 filter 。filter 常出现在 bool 子句，constant_score 子句和 filter 聚合查询中。

一个关于 query 和 filter 的例子：

```json
GET /_search
{
  "query": { 
    "bool": { 
      "must": [
        { "match": { "title":   "Search"        }},
        { "match": { "content": "Elasticsearch" }}
      ],
      "filter": [ 
        { "term":  { "status": "published" }},
        { "range": { "publish_date": { "gte": "2015-01-01" }}}
      ]
    }
  }
}
```

## 复合查询

### bool 子句

bool 有四种子句，分别是：must、filter、must_not、should

- must：查询内容必须出现在文档中，匹配成功则加分
- filter：与 must 一样，不过不计算得分 
- should：查询内容可以出现，也可以不出现，但是一旦出现，会加分
- must_not：查询内容一定不出现，不计算得分，是一种 filter

例子：

```json
POST _search
{
  "query": {
    "bool" : {
      "must" : {
        "term" : { "user" : "kimchy" }
      },
      "filter": {
        "term" : { "tag" : "tech" }
      },
      "must_not" : {
        "range" : {
          "age" : { "gte" : 10, "lte" : 20 }
        }
      },
      "should" : [
        { "term" : { "tag" : "wow" } },
        { "term" : { "tag" : "elasticsearch" } }
      ],
      "minimum_should_match" : 1,
      "boost" : 1.0
    }
  }
}
```

文档的最终得分就是 must 中的分数加上 should 中的分数。

minimum_should_match的意思是 should 中必须匹配到的子句数量，如果 bool 中至少包含一个 should ，并且没有 must 和 filter，那这个值默认就是1，如果含有 must 或 filter 的话，默认就是0.

filter 匹配到文档得分为0，如果 must 中的查询语句为 match_all ，则每个文档的得分为1.0:

```json
GET _search
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "term": {
          "status": "active"
        }
      }
    }
  }
}
```

上面这个查询和 constant_score 子句实现的效果是一模一样的，得分都是1.0：

```json
GET _search
{
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "status": "active"
        }
      }
    }
  }
}
```



### boosting 子句

boosting 有三个子句：positive、negative、和 negative_boost

positive 表示必须匹配到才返回文档。

negative 中的子句如果匹配到了 positive 中返回的文档，则对文档分数进行计算，计算方式是原始得分乘以 negative_boost 的分数

下边是实例：

```json
GET /_search
{
    "query": {
        "boosting" : {
            "positive" : {
                "term" : {
                    "text" : "apple"
                }
            },
            "negative" : {
                 "term" : {
                     "text" : "pie tart fruit crumble tree"
                }
            },
            "negative_boost" : 0.5
        }
    }
}
```

negative_boost 的值位于 0 到 1 之间。

### constant_score 子句

constant_score 子句用于固定得分，主要包含两个子句：filter、boost

意思就是 filter 匹配到的所有子句都是 boost 指定的分数，boost 默认为 1.0：

```json
GET /_search
{
    "query": {
        "constant_score" : {
            "filter" : {
                "term" : { "user" : "kimchy"}
            },
            "boost" : 1.2
        }
    }
}
```



### dis_max 子句

dis_max 可以实现 best fields 策略多字段搜索。

什么意思那，首先来个例子，加入以下数据到ES。

```json
POST /forum/_bulk?pretty&refresh
{ "index": { "_id": "1"} }
{ "doc" : {"title": "this is elasticsearch blog", "content" : "i like to write best elasticsearch article"} }
{ "index": { "_id": "2"} }
{ "doc" : {"content" : "i think java is the best programming language", "title": "this is java blog"} }
{ "index": { "_id": "3"} }
{ "doc" : {"content" : "i am only an elasticsearch beginner", "title": "this is elasticsearch blog"} }
{ "index": { "_id": "4"} }
{ "doc" : {"content" : "elasticsearch and hadoop are all very good solution, i am a beginner", "title": "this is java, elasticsearch, hadoop blog"} }
{ "index": { "_id": "5"} }
{ "doc" : {"content" : "spark is best big data solution based on scala ,an programming language similar to java", "title": "this is spark blog"} }
```

然后执行一个 bool 子句查询：

```json
GET /forum/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "doc.title": "java solution"
          }
        }, {
          "match": {
            "doc.content": "java solution"
          }
        }
      ]
    }
  }
}
```

这是的返回结果是：

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
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 1.8488126,
    "hits" : [
      {
        "_index" : "forum",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 1.8488126,
        "_source" : {
          "doc" : {
            "content" : "i think java is the best programming language",
            "title" : "this is java blog"
          }
        }
      },
      {
        "_index" : "forum",
        "_type" : "_doc",
        "_id" : "4",
        "_score" : 1.5563383,
        "_source" : {
          "doc" : {
            "content" : "elasticsearch and hadoop are all very good solution, i am a beginner",
            "title" : "this is java, elasticsearch, hadoop blog"
          }
        }
      },
      {
        "_index" : "forum",
        "_type" : "_doc",
        "_id" : "5",
        "_score" : 1.4233949,
        "_source" : {
          "doc" : {
            "content" : "spark is best big data solution based on scala ,an programming language similar to java",
            "title" : "this is spark blog"
          }
        }
      }
    ]
  }
}
```

可以看到，title 和 content 都匹配到单词的得分更高，而 id=5 的 content 中，java 和  solution 都存在，但是得分却低。

bool 语句的分数生成策略是：

（第一条得分 + 第二条得分）* 匹配的条数 / 总条数

注意这里 `匹配的条数` ，因为 id=5 的文档中，title 中并没有匹配到关键词，所以在相乘中很吃亏。

那么这种情况下，使用 dis_max 子句就可以解决这个问题：

```json
GET /forum/_search
{
  "query": {
    "dis_max": {
      "queries": [
          {
            "match": {
              "doc.title": "java solution"
            }
          },{
          "match": {
            "doc.content": "java solution"
          }
        }
      ]
    }
  }
}
```

其结果为：

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
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 1.4233949,
    "hits" : [
      {
        "_index" : "forum",
        "_type" : "_doc",
        "_id" : "5",
        "_score" : 1.4233949,
        "_source" : {
          "doc" : {
            "content" : "spark is best big data solution based on scala ,an programming language similar to java",
            "title" : "this is spark blog"
          }
        }
      },
      {
        "_index" : "forum",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 0.9395274,
        "_source" : {
          "doc" : {
            "content" : "i think java is the best programming language",
            "title" : "this is java blog"
          }
        }
      },
      {
        "_index" : "forum",
        "_type" : "_doc",
        "_id" : "4",
        "_score" : 0.7942397,
        "_source" : {
          "doc" : {
            "content" : "elasticsearch and hadoop are all very good solution, i am a beginner",
            "title" : "this is java, elasticsearch, hadoop blog"
          }
        }
      }
    ]
  }
}
```

dis_max 的策略很简单，直接取做高分数的 query。

dis_max 还可以加一个 tie_breaker 子句，用于平衡两者，它默认是0，所以默认时直接取最高 query 的分数：

```json
GET /forum/_search
{
  "query": {
    "dis_max": {
      "queries": [
          {
            "match": {
              "doc.title": "java solution"
            }
          },{
          "match": {
            "doc.content": "java solution"
          }
        }
      ],
      "tie_breaker" : 0.7
    }
  }
}
```



加上 tie_breaker 后，策略如下：

1. 取最高query的分数
2. 将其他每一个query的分数乘以 tie_breaker 
3. 将最高 query 的分数加上第二步中计算得到的每一个值

### function_score 子句

function_score 子句可以自定义文档得分！例如：

```json
GET /_search
{
    "query": {
        "function_score": {
            "query": { "match_all": {} },
            "boost": "5",
            "random_score": {}, 
            "boost_mode":"multiply"
        }
    }
}
```

