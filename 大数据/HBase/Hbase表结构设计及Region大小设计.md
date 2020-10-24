# HBase 表结构设计及 Region 大小设计

官方文档：https://hbase.apache.org/book.html#schema

官方文档：https://hbase.apache.org/book.html#regionserver_sizing_rules_of_thumb

进行ColumnFamily修改时，必须禁用表，例如：

```java
Configuration config = HBaseConfiguration.create();
Admin admin = new Admin(conf);
TableName table = TableName.valueOf("myTable");

admin.disableTable(table);

HColumnDescriptor cf1 = ...;
admin.addColumn(table, cf1);      // adding new ColumnFamily
HColumnDescriptor cf2 = ...;
admin.modifyColumn(table, cf2);    // modifying existing ColumnFamily

admin.enableTable(table);
```

当对Tables或ColumnFamilies进行更改（例如，region大小，块大小）时，这些更改将在下次进行重大压缩时生效，并且 StoreFiles 将被重写。 有关StoreFiles的更多信息，请参见[store](https://hbase.apache.org/book.html#store) 。



## 表架构的经验法则

有许多不同的数据集，具有不同的访问模式和服务级别期望。因此，这些经验法则只是一个概述。阅读完此列表后，请阅读本章的其余部分以获取更多详细信息。

- regions 的大小在10到50 GB之间。
- cell 不超过10 MB，如果使用mob，则不超过50 MB。否则，请考虑将单元数据存储在HDFS中，并将指向数据的指针存储在HBase中。
- 典型的架构每个表具有1到3个列族。 HBase表不应设计为模仿RDBMS表。
- 对于带有1或2个列族的表，大约50-100个 regions 是一个不错的数字。请记住，regions 是列族的连续段。
- 列的限定符应尽量短。它们不应该像典型的RDBMS中那样具有自我说明性和描述性。
- 如果要存储基于时间的机器数据或日志记录信息，并且行键是基于设备ID或服务ID加上时间的，则您可能会遇到一种模式，在这种模式下，较旧的数据区域在特定使用期限内永远不会进行其他写入操作。在这种情况下，最终会出现少量活动 regions 和大量较旧的 regions，而这些 regions 没有新的写入操作。对于这些情况，您可以容忍更多区域，因为资源消耗仅由活动 regions 驱动。
- 如果只有一个列族忙于写操作，则只有该列族会占用内存。分配资源时要注意写模式。



## RegionServer大小调整规则

> 就个人而言，除非您的工作负载非常繁重，否则我将把每台机器可以为HBase专门提供的最大磁盘空间放在6T左右。在这种情况下，Java堆应为32GB（20G区域，128M内存存储，其余为默认值）。

## 关于列族的数量

HBase当前不能很好地处理超过两个或三个列族的任何事物，因此请保持架构中有尽量少的列族。当前，刷新是在每个 regions 的基础上进行的，因此，如果一个列族承载着大量要进行刷新的数据，即使相邻族族携带的数据量很小，也将对其进行刷新。当存在许多列族时，冲洗交互可以产生一堆不必要的I / O（通过更改冲洗以在每个列系列的基础上解决）。此外，在表/区域级别触发的压缩也将在每个 store 中发生。

如果可以，请尝试使用一个列族。仅在通常以列为范围的数据访问的情况下才引入第二和第三列族。也就是说，您查询一个列族或另一个列族，但通常一次都不查询。

#### ColumnFamilies的行数

在单个表中存在多个 ColumnFamilies 的地方，请注意行数。如果ColumnFamilyA有100万行，而ColumnFamilyB有10亿行，则ColumnFamilyA的数据可能会分布在许多RegionServers。这使得对ColumnFamilyA进行批量扫描的效率较低。



## Rowkey 设计



































