# ES 常用操作

搜索全部：

```http
GET _search
{
  "query": {
    "match_all": {}
  }
}
```

查看节点并按名称排序：

```http
GET /_cat/nodes?s=name&v
```

查看磁盘使用量：

```http
GET /_cat/allocation?v
```

创建索引：

```http
PUT /test?pretty
```

查看索引的设置：

```http
GET /test/_setting?pretty
```

创建索引时设置分片和副本：

```http
PUT /test-1/?pretty
{
  "setting": {
     "number_of_shards": 15,
     "number_of_replicas": 5
  }
}
```

查看分片分布：

```http
GET /_cat/shards/test-1?s=shard&v
```

查看文档数量：

```http
GET /_cat/count?v
```

某一索引的文档数量：

```http
GET /_cat/count/sessions2-200803?v
```

查看索引列表，按大小排序：

```http
GET /_cat/indices?s=store.size:desc&v
```



为将来的索引设置默认分片和副本 ：

```http
PUT /_template/log
{
  "index_patterns": "session2-*",
  "settings": {
    "number_of_shards": 6,
    "number_of_replicas": 1
  }
}
```

分片数量是不能修改的，但是可以修改副本数量：

```http
PUT /sessions2-200804/_settings
{
  "index" : {
    "number_of_replicas" : 0
  }
}
```

## 开启慢查询日志

```http
PUT /_all/_settings?preserve_existing=true
{
  "index.search.slowlog.threshold.fetch.warn": "4s",
  "index.search.slowlog.threshold.query.warn": "4s"
}
```









