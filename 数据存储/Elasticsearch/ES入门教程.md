# ES入门教程

首先，要先安装 ES 及 Kibana ，这两者是非常易于安装的，无论是单机还是集群环境，比其他任何分布式软件都要好安装。我使用的 ES 即 Kibana 版本是 7.5.0。

## 一些语句

首先，先查询一下所有数据：

```json
GET _search
{
  "query": {
    "match_all": {}
  }
}
```

因为还没有注入数据，所以看到的内容只是8条 Kibana 相关的数据。

然后可以插入一些数据，这里的 customer 就是 index，_doc 是 type ，1 是 id，需要注意，type 必须是 _doc，不能省略：

```json
PUT /customer/_doc/4
{
  "name": "xizhihao",
  "age": 22,
  "widget": 60,
  "height": 172
}
```

这里使用的是 PUT 请求，PUT 请求必须将 index，type，id全部写上，Kibana 中，最前面的 / 可以省略，这里也可以是 POST 请求，如果是 POST 请求的话，后面的 id 可以省略，ES 会自动给文档生成一个 id 。

下面是返回的结果：

```json
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "4",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 4,
  "_primary_term" : 1
}
```

这里，_index 是索引，ES 中所有数据都是储存在索引之下的， _type 是类型，不过新版的 ES 已弃用这个字段，这里必须固定为 _doc。 _id 就是 id，可以自定义。 _version 是文档的版本，如果版本是 1，下面的 result 就是 created ，这个 version 就是文档的版本，可以标记文档被修改了多少次了。 _shards 是分片信息。 _seq_no 是自增的，每个写入操作都会自增。 _primary_term会在恢复数据时，比如重启时进行自增。

## 查询数据

查询所有数据：

```json
GET _search
{
  "query": {
    "match_all": {}
  }
}
```

这是查询的所有数据，但是只显示了10条，可以加个 size 字段来自定义查询数量：

```json
GET _search
{
  "query": {
    "match_all": {}
  },
  "size": 20
}
```

下面是我查询的结果，前面的是一些统计信息，在两层 hits 之后就是真正的文档了，我们自己的数据放在了文档的 _source 字段中了，关于这些 ES 自己生成的字段的含义，可以看官方文档：https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html

```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 16,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "customer",
        "_type" : "_doc",
        "_id" : "4",
        "_score" : 1.0,
        "_source" : {
          "name" : "xizhihao",
          "age" : 22,
          "widget" : 60,
          "height" : 172
        }
      },
      {
        "_index" : "customer",
        "_type" : "_doc",
        "_id" : "5",
        "_score" : 1.0,
        "_source" : {
          "name" : "suning",
          "age" : 47,
          "widget" : 60,
          "height" : 172
        }
      },
      {
        "_index" : "customer",
        "_type" : "_doc",
        "_id" : "r0ObEW8BbWWfcviXHjjN",
        "_score" : 1.0,
        "_source" : {
          "name" : "anbang",
          "age" : 22,
          "widget" : 70,
          "height" : 170
        }
      }
    ]
  }
}
```



然后也可以根据 index ，type，id 查询单个文档，这里 type 必须为 _doc：

```
GET /customer/_doc/1
```

返回的数据：

```json
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 3,
  "_seq_no" : 8,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "xujiyou",
    "age" : 22,
    "widget" : 68,
    "height" : 175
  }
}
```

## Bulk API

Bulk API 用于往 ES 一次性插入一堆数据，要求的数据格式是这样的：

```json
{"index":{"_id":"1"}}
{"account_number":1,"balance":39225,"firstname":"Amber","lastname":"Duke","age":32,"gender":"M","address":"880 Holmes Lane","employer":"Pyrami","email":"amberduke@pyrami.com","city":"Brogan","state":"IL"}
{"index":{"_id":"6"}}
{"account_number":6,"balance":5686,"firstname":"Hattie","lastname":"Bond","age":36,"gender":"M","address":"671 Bristol Street","employer":"Netagy","email":"hattiebond@netagy.com","city":"Dante","state":"TN"}
```

前面一个index json，后面跟一个具体数据的 json

数据地址：https://raw.githubusercontent.com/elastic/elasticsearch/master/docs/src/test/resources/accounts.json

然后用下面的 API 进行储存进 bank index中：

```shell
curl -H "Content-Type: application/json" -XPOST "localhost:9200/bank/_bulk?pretty&refresh" --data-binary "@/Users/jiyouxu/Downloads/accounts.json"
```

再用下面的 API 查询 index :

```bash
curl "localhost:9200/_cat/indices?v"
```

## 一些查询语句

既然插入了一些数据，下面就来看一下怎么查询吧。

查询的 endpoint 是 `_search` ，如果要是使用 ES 全套的查询服务，可以使用 `Query DSL`。

下面的例子查询 bank 中的所有文档:

```json
GET /bank/_search
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ]
}
```

下面是结果：

```json
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1000,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "0",
        "_score" : null,
        "_source" : {
          "account_number" : 0,
          "balance" : 16623,
          "firstname" : "Bradshaw",
          "lastname" : "Mckenzie",
          "age" : 29,
          "gender" : "F",
          "address" : "244 Columbus Place",
          "employer" : "Euron",
          "email" : "bradshawmckenzie@euron.com",
          "city" : "Hobucken",
          "state" : "CO"
        },
        "sort" : [
          0
        ]
      },
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : null,
        "_source" : {
          "account_number" : 1,
          "balance" : 39225,
          "firstname" : "Amber",
          "lastname" : "Duke",
          "age" : 32,
          "gender" : "M",
          "address" : "880 Holmes Lane",
          "employer" : "Pyrami",
          "email" : "amberduke@pyrami.com",
          "city" : "Brogan",
          "state" : "IL"
        },
        "sort" : [
          1
        ]
      },
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "9",
        "_score" : null,
        "_source" : {
          "account_number" : 9,
          "balance" : 24776,
          "firstname" : "Opal",
          "lastname" : "Meadows",
          "age" : 39,
          "gender" : "M",
          "address" : "963 Neptune Avenue",
          "employer" : "Cedward",
          "email" : "opalmeadows@cedward.com",
          "city" : "Olney",
          "state" : "OH"
        },
        "sort" : [
          9
        ]
      }
    ]
  }
}

```

还可以分页搜索，这通过 from 和 size 来实现：

```json
GET /bank/_search
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ],
  "from": 10,
  "size": 10
}
```

上面的搜索都是 match_all ，匹配所有，下面来看下怎么搜索某个字段中的数据：

```json
GET /bank/_search
{
  "query": { "match": { "address": "mill lane" } }
}
```

将 match 替换 match_all ，然后 match里面填写需要查询的字段，及字段内包含的值。注意，这里 mill lane 不是完全匹配，而是分词匹配，匹配到 mill 或 lane 都会成功。

但就想完全匹配 “mill lane” 怎么办！那就需要用 match_phrase 了：

```json
GET /bank/_search
{
  "query": { "match_phrase": { "address": "mill lane" } }
}
```

下面构造一个更复杂的查询，可以使用 bool 来组合多个查询语句：

```json
GET /bank/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "age": "40" } }
      ],
      "must_not": [
        { "match": { "state": "ID" } }
      ]
    }
  }
}
```

bool 子句中可以包含 must，must_not，should。must 和 should 会影响 文档的得分，默认情况下，ES 按照得分来排序文档。

must_not 子句中的条件被视为筛选器，它影响文档是否包含在结果中，而不影响得分。

当然也可以指定其他过滤器，不仅仅是逻辑非：

```json
GET /bank/_search
{
  "query": {
    "bool": {
      "must": { "match_all": {} },
      "filter": {
        "range": {
          "balance": {
            "gte": 20000,
            "lte": 30000
          }
        }
      }
    }
  }
}
```

## 使用聚合

聚合的作用就是用于数据分析，比如总共有多少，平均是多少。

```json
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      }
    }
  }
}
```

这里设置 size 为 0 的意思是只显示聚合数据，不显示文档数据。

group_by_state 是聚合的名字，会返回在结果中，

terms 用于指定聚合的具体内容，field 代表要匹配字段名，state 就是字段名，keyword 代表没有分词，指定了字段名的类型。

下面是结果：

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
      "value" : 1000,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "group_by_state" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 743,
      "buckets" : [
        {
          "key" : "TX",
          "doc_count" : 30
        },
        {
          "key" : "MD",
          "doc_count" : 28
        },
        {
          "key" : "ID",
          "doc_count" : 27
        },
        {
          "key" : "AL",
          "doc_count" : 25
        },
        {
          "key" : "ME",
          "doc_count" : 25
        },
        {
          "key" : "TN",
          "doc_count" : 25
        },
        {
          "key" : "WY",
          "doc_count" : 25
        },
        {
          "key" : "DC",
          "doc_count" : 24
        },
        {
          "key" : "MA",
          "doc_count" : 24
        },
        {
          "key" : "ND",
          "doc_count" : 24
        }
      ]
    }
  }
}
```

也可以进行嵌套聚合，先对 state 进行聚合，然后对聚合到的数据取平均数。：

```json
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
```

也可以对数量进行排序：

```json
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword",
        "order": {
          "average_balance": "desc"
        }
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
```

好了，入门教程就先到这里，主要参考的是官方入门文档：[Getting started with Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started.html)

后面再学习其他文档。

需要重点学习的内容：聚合、Query DSL、Mapping、REST APIs。最重要的就这四个部分！