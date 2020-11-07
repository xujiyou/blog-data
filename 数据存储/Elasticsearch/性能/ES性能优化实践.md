# ES 性能优化实践

## 数据预加载

15 个 ES 节点，每个节点的 JVM 堆内存有 32GB，一共有 13亿条数据，共 690GB，分了 16个分片，一个分片 46GB，但是在查询时，发现虽然内存很多，但是查询并不快，一个查询要 19 秒。

参考：https://blog.csdn.net/miaomiao19971215/article/details/105487619



Elasticsearch在启动时会打开并读取硬盘上的部分index segment文件，并缓存数据至内存中，后续的搜索操作都会在内存中进行。如果待搜索的数据不在内存中，则会打开相应的index segment文件，并读取数据至内存。这种预加载的做法有助于提高Elasticsearch对外提供服务的响应速度，毕竟减少了打开index semgent的IO操作次数。

有两种方案能设置预加载中需要打开的segment文件:

1. 【全局配置】 在config/elasticsearch.yml中添加index.store.preload，这是一个数组，里面存放了需要预加载的文件的后缀。后缀的类别和含义如下:
   1. nvd: 该文件中存储了影响相关度分数的因素。
   2. dvd: 存储了文档的数据。
   3. tim: 文档字典。
   4. doc: 发布清单
   5. dim: 点数据
      支持通配符\*，如果写成["*"]，则可以将所有的数据全部缓存到内存。

```bash
index.store.preload: ["nvd", "dvd"]
```

1. 【局部配置】在创建index时，指定该index需要预加载的文件。注意，不得在创建完index后更新(新增)index.store.preload，否则报错:
   Can’t update non dynamic settings… index.store.preload属于静态配置。

```bash
PUT /my_index
{
  "settings": {
    "index.store.preload": ["nvd", "dvd"]
  }
}
```

有些人尝到甜头后，打算在预加载时尽可能的把热门数据放到内存中，但值得注意的是，数据预加载也有其缺陷。如果预加载的数据量过大，比如几乎占满了服务器分配给Elasticsearch的最大堆内存，而后续进行冷门数据搜索时，搜索的内容恰好不在内存中，则不得不从Disk上打开index segment，此时为了避免内存溢出，ES会丢弃一部分旧的数据，断开部分index_segement文件的IO流，腾出空间存储新的数据，那么热门数据就丢失了。而热门数据的查询概率比冷门数据大得多，导致后续搜索数据时，需要进行更多的IO操作，这样反而得不偿失。



## 内存分配

32 GB 是 ES 一个内存设置限制，那如果你的机器有很大的内存怎么办呢？现在的机器内存普遍都大，一般都有 300 - 500 GB 内存的机器。

当然，如果有这种机器，那是极好的，接下来有两个方案：

- 如果主要做全文检索，可以考虑给 Elasticsearch 32 G 内存，剩下的交给 Lucene 用作操作系统的文件系统缓存，所有的 segment 都缓存起来，会加快全文检索。
- 如果需要更多的排序和聚合，那就需要更大的堆内存。可以考虑一台机器上创建两个或者更多的 ES 节点，而不要部署一个使用 32 + GB 内存的节点。仍然要坚持 50% 原则，假设你有个机器有 128 G 内存，你可以创建两个 node，使用 32 G 内存。也就是说 64 G 内存给 ES 的堆内存，剩下的 64 G 给 Lucene。

PS：如果选择第二种方案，需要配置 `cluster.routing.allocation.same_shard.host: true`。这会防止同一个 shard 的主副本存在同一个物理机上（因为如果存在一个机器上，副本的高可用性就没有了）。



## 命中

ES 进行查询时，返回的 total 数量即为命中数量。

如果搜索命中的条数少，查询速度就快。命中条数与查询速度成正相关。

可以在查询时设置 `terminate_after` 限制命中数量，当到达设置的命中数量时可以直接返回，防止命中过多查询时间无限大。

参考：https://www.elastic.co/guide/en/elasticsearch/reference/6.8/search-request-body.html



## ES写入优化

https://www.easyice.cn/archives/207

































