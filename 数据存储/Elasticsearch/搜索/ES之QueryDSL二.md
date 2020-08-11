# ES 之 Query DSL 二

这篇文章来学习 ES 的全文检索，可以学习如何检索一篇文章的任意内容。在索引过程中，ES 会使用分词器对文档进行分词处理。

## intervals 查询

首先先来一个例子，先插入两条数据：

```json
POST article/_doc/1
{
  "content": "my favorite food is cold porridge but not when it's cold my favorite food is porridge"
}

POST article/_doc/2
{
  "content": "my favorite food is cold porridge but not when it's cold my favorite food is hot water"
}
```

然后使用 intervals 进行查询:

```json
POST article/_search
{
  "query": {
    "intervals" : {
      "content" : {
        "all_of" : {
          "ordered" : true,
          "intervals" : [
            {
              "match" : {
                "query" : "my favorite food",
                "max_gaps" : 0,
                "ordered" : true
              }
            },
            {
              "any_of" : {
                "intervals" : [
                  { "match" : { "query" : "hot water" } },
                  { "match" : { "query" : "cold porridge" } }
                ]
              }
            }
          ]
        }
      }
    }
  }
}
```

结果：

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
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 0.5,
    "hits" : [
      {
        "_index" : "article",
        "_type" : "_doc",
        "_id" : "2",
        "_score" : 0.5,
        "_source" : {
          "content" : "my favorite food is cold porridge but not when it's cold my favorite food is hot water"
        }
      },
      {
        "_index" : "article",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.3333333,
        "_source" : {
          "content" : "my favorite food is cold porridge but not when it's cold my favorite food is porridge"
        }
      }
    ]
  }
}

```



可以看到 ，intervals 下面跟的是文档的字段名称，随后跟一个 all_of ，这里除了 all_of，还可以跟 match 、prefix、wildcard、any_of、filter。

每种匹配模式下面使用的子句是不一样的，下面分别来介绍。

### all_of

all_of 是其他 interval 的组合，它包含以下几个子句：

- **intervals** interval 的组合数组
- **max_gaps** 默认是 -1，关键字之间的最大间隔，如果为 0，则关键字必须相邻出现。
- **ordered** 默认为 false，如果为 true 的话那，就是关键字在文章中越靠前越好！
- **filter** 过滤规则

### match

- **query** 查询的内容。
- **max_gaps** 上同
- **ordered** 上同
- **analyzer** 指定分词器
- **filter** 过滤规则
- **use_field** 指定要搜索的文档字段，而不单单是顶级字段

### prefix

前缀匹配，匹配一个单词的前缀，比如 favorite ，可以前缀搜索 favo，最多128个字母

- **prefix** 要搜索的前缀
- **analyzer** 指定分词器
- **use_field** 搜索的字段

### wildcard

通配符匹配，最多128个字母

- **pattern** 通配符，？可代替一个字符，*可代替 0 或多个字符，注意，以 * 或 ?. 开头会降低性能。
- **analyzer** 指定分词器
- **use_field** 搜索的字段

### any_of

任何一个子模式被匹配了都返回

- **intervals** 模式数组
- **filter** 过滤器

### filter

过滤器，不影响分数。

- **after** 在模式匹配之后执行
- **before** 在模式匹配之前执行
- **contained_by** 包含
- **containing** 包含
- **not_contained_by** 不包含
- **not_containing** 不包含
- **not_overlapping** 不重叠
- **overlapping** 重叠
- **script** 脚本过滤，在脚本中可以使用变量 `interval` 它包含三个属性：`start` `end` `gaps`

## match 查询

匹配查询是执行全文搜索的标准查询，包括模糊匹配选项。

先看例子：

```json
GET /_search
{
    "query": {
        "match" : {
            "message" : {
                "query" : "this is a test"
            }
        }
    }
}
```

match 下面是一个文档字段，这里的字段是 message，字段下面包含以下子句

- **query** 想要查询的文本，数字，日期，或bool值
- **analyzer** 分词器
- **auto_generate_synonyms_phrase_query** 默认为 true，自动生成同义词
- **fuzziness** 匹配距离
- **max_expansions** 查询将要扩展的最大数量，默认50
- **prefix_length**  默认为0，模糊查询不能改变的前缀长度
- **transpositions** 默认为 true，模糊匹配包括将两个相邻的字符转置，如（ab → ba），可防止拼写错误
- **fuzzy_rewrite** match 子句可以修改为多种形式，参照：[`rewrite` parameter](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-term-rewrite.html)
- **lenient** 默认为 false，为true时，会考虑字段的类型
- **operator** 操作，有两种` OR` `AND` 默认是 OR
- **minimum_should_match** 至少应该匹配几个
- **zero_terms_query** 两个值 `none` `all` 默认是 none



## match_bool_prefix 查询

示例：

```json
GET /_search
{
    "query": {
        "match_bool_prefix" : {
            "message" : "quick brown f"
        }
    }
}
```

相当于：

```json
GET /_search
{
    "query": {
        "bool" : {
            "should": [
                { "term": { "message": "quick" }},
                { "term": { "message": "brown" }},
                { "prefix": { "message": "f"}}
            ]
        }
    }
}
```

match_bool_prefix 又一个子句为 analyzer ，可以指定分词器。

## match_phrase 查询

match_phrase 是短语查询，类似于上面的 intervals 的 max_gaps 为 0。

示例：

```json
GET /_search
{
    "query": {
        "match_phrase" : {
            "message" : "this is a test"
        }
    }
}
```

## match_phrase_prefix 查询

match_bool_prefix 和 match_phrase 的联合版：

```json
GET /_search
{
    "query": {
        "match_phrase_prefix" : {
            "message" : {
                "query" : "quick brown f"
            }
        }
    }
}
```

## multi_match 查询

match 只能查一个字段，multi_match 可以查询多个字段

```json
GET /_search
{
  "query": {
    "multi_match" : {
      "query":    "this is a test", 
      "fields": [ "subject", "message" ] 
    }
  }
}
```

字段名也可以使用通配符

```json
GET /_search
{
  "query": {
    "multi_match" : {
      "query":    "Will Smith",
      "fields": [ "title", "*_name" ] 
    }
  }
}
```

如果 fields 为空，则将对所有字段进行搜索

好了，到此为止，所有的查询就都有涉及了。后面的 Common term Query 在 7.3.0 版本中标明过期了，而Query String 和 Simple Query String 可能会带来注入，所以不学习了，关于全文索引，前面介绍的完全够用。