# HBase 官方架构

官方文档：https://hbase.apache.org/book.html#_architecture

HBase是一种“ NoSQL”数据库。 “ NoSQL”是一个通用术语，表示该数据库不是支持SQL作为其主要访问语言的RDBMS，但是NoSQL数据库的类型很多：BerkeleyDB是本地NoSQL数据库的一个示例，而HBase在很大程度上分布式数据库。从技术上讲，HBase实际上比“数据库”更像是“数据存储”，因为它缺少您在RDBMS中发现的许多功能，例如类型列，二级索引，触发器和高级查询语言等。

但是，HBase具有支持线性和模块化缩放的许多功能。 HBase群集通过添加在商品类服务器上托管的RegionServer来扩展。例如，如果群集从10个RegionServer扩展到20个，则在存储和处理能力方面都会翻倍。 RDBMS可以很好地扩展，但只能扩展到一个点-特别是单个数据库服务器的大小-为了获得最佳性能，需要专用的硬件和存储设备。注意的HBase功能包括：

- 高度一致的读/写：HBase不是“最终一致”的数据存储。这使得它非常适合诸如高速计数器聚合之类的任务。
- 自动分片：HBase表通过区域分布在群集上，并且随着数据的增长，区域会自动拆分和重新分布。
- 自动RegionServer故障转移
- Hadoop / HDFS集成：HBase开箱即用地支持HDFS作为其分布式文件系统。
- MapReduce：HBase支持通过MapReduce大规模并行化处理，以将HBase用作源和接收器。
- Java客户端API：HBase支持易于使用的Java API进行编程访问。
- Thrift / REST API：HBase还为非Java前端支持Thrift和REST。
- 块缓存和Bloom过滤器：HBase支持块缓存和Bloom过滤器，以进行大量查询优化。
- 运维管理：HBase提供了内置的网页，以提供运营见解以及JMX指标。

---

HBase并非适合所有问题。

首先，请确保您有足够的数据。如果您有数亿或数十亿行，那么HBase是一个很好的选择。如果您只有数千行/百万行，则使用传统的RDBMS可能是一个更好的选择，原因是您的所有数据都可能在单个节点（或两个节点）上结束，而集群的其余部分可能处于停顿状态闲。

其次，确保没有RDBMS提供的所有其他功能（例如，类型化的列，二级索引，事务，高级查询语言等），您就可以生存。不能简单地通过更改将基于RDBMS构建的应用程序“移植”到HBase。例如JDBC驱动程序。考虑从RDBMS迁移到HBase，作为与端口相对的完整重新设计。

第三，确保您有足够的硬件。甚至少于5个数据节点（由于诸如HDFS块复制（默认值为3）之类的东西）以及一个NameNode的情况，HDFS也不能很好地完成工作。

HBase在笔记本电脑上可以很好地独立运行-但这仅应视为开发配置。

---

HDFS是一个分布式文件系统，非常适合存储大文件。它的文档指出，它不是通用文件系统，并且不提供文件中的快速个人记录查找。另一方面，HBase构建在HDFS之上，并为大型表提供快速记录查找（和更新）。有时这可能是概念上的混乱点。 HBase在内部将数据放入HDFS上存在的索引“ StoreFiles”中，以进行高速查找。有关HBase如何实现其目标的更多信息，请参见数据模型和本章的其余部分。



## 目录表

目录表 `hbase:meta` 作为 HBase 表存在，并已从 HBase Shell 的 list 命令中过滤掉，但实际上是一个表，与其他任何表一样。



#### hbase:meta

`hbase:meta`表（以前的版本叫 .META.），保存了 HBase 中的所有 regions，并且 hbase:meta 的位置是储存在 ZooKeeper 中的 。

hbase:meta 的结构如下：

**Key**

- Key 的格式为 `[table],[region start key],[region id]`

**Values**

- `info:regioninfo`（[HRegionInfo](https://hbase.apache.org/apidocs/org/apache/hadoop/hbase/HRegionInfo.html) 实例的序列化）
- `info:server`（RegionServer 的地址信息）
- `info:serverstartcode`（RegionServer 的启动时间）

当表正在拆分时，将创建另外两个列，分别为`info:splitA`和 `info:splitB` 。这些列代表两个子区域。这些列的值也是序列化的HRegionInfo实例。分割区域后，最终将删除此行。

运行以下命令查看这个表：

```
hbase(main)> scan 'hbase:meta'
```



















