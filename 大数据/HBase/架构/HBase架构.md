# HBase 架构

这篇文章挺好的：https://zhuanlan.zhihu.com/p/30414252

物理上看, HBase系统有3种类型的后台服务程序, 分别是Region server, Master server 和 zookeeper.

Region server负责实际数据的读写. 当访问数据时, 客户端与HBase的Region server直接通信.

Master server管理Region的位置, DDL(新增和删除表结构)

Zookeeper负责维护和记录整个HBase集群的状态.

所有的HBase数据都存储在HDFS中. 每个 Region server都把自己的数据存储在HDFS中. 如果一个服务器既是Region server又是HDFS的Datanode. 那么这个Region server的数据会在把其中一个副本存储在本地的HDFS中, 加速访问速度.



## HBase Region server

HBase的表根据Row Key的区域分成多个Region, 一个Region包含这这个区域内所有数据. 而Region server负责管理多个Region, 负责在这个Region server上的所有region的读写操作. 一个Region server最多可以管理1000个region.



## HBase Master server

HBase Maste主要负责分配region和操作DDL(如新建和删除表)等,

HBase Master的功能:

- 协调Region server
- 在集群处于数据恢复或者动态调整负载时,分配Region到某一个Region Server中
- 管控集群, 监控所有RegionServer的状态
- 提供DDL相关的API, 新建(create),删除(delete)和更新(update)表结构.





## HBase的第一次读写流程

HBase把各个region的位置信息存储在一个特殊的表中, 这个表叫做Meta table.

Zookeeper里面存储了这个Meta table的位置信息.

HBase的访问流程:

1. 客户端访问Zookeep, 得到了具体Meta table的位置

2. 客户端再访问真正的Meta table, 从Meta table里面得到row key所在的region server

3. 访问rowkey所在的region server, 得到需要的真正数据.

客户端缓存meta table的位置和row key的位置信息, 这样就不用每次访问都读zookeeper.

如果region server由于宕机等原因迁移到其他服务器. Hbase客户端访问失败, 客户端缓存过期, 再重新访问zookeeper, 得到最新的meta table位置, 更新缓存.



## HBase Meta Table

Meta table存储所有region的列表

Meta table用类似于Btree的方式存储

Meta table的结构如下:

- Key: region的开始row key, region id

- Values: Region server



## Region Server的结构

Region Server运行在HDFS的data node上面, 它有下面4个部分组成:

- WAL: 预写日志(Write Ahead Log)是一HDFS上的一个文件, 如果region server崩溃后, 日志文件用来恢复新写入的的, 但是还没有存储在硬盘上的数据.
- BlockCache: 读取缓存, 在内存里缓存频繁读取的数据, 如果BlockCache满了, 会根据LRU算法(Least Recently Used)选出最不活跃的数据, 然后释放掉
- MemStore: 写入缓存, 在数据真正被写入硬盘前, Memstore在内存中缓存新写入的数据. 每个region的每个列簇(column family)都有一个memstore. memstore的数据在写入硬盘前, 会先根据key排序, 然后写入硬盘.
- HFiles: HDFS上的数据文件, 里面存储KeyValue对.