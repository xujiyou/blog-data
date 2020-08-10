# ES 性能优化建议



## 磁盘建议

Elasticsearch部署建议：选择合理的硬件配置，尽可能使用SSD

Elasticsearch最大的瓶颈往往是磁盘读写性能，尤其是随机读取性能。使用SSD（PCI-E接口SSD卡/SATA接口SSD盘）通常比机械硬盘（SATA盘/SAS盘）查询速度快5~10倍，写入性能提升不明显。

对于文档检索类查询性能要求较高的场景，建议考虑SSD作为存储，同时按照1:10的比例配置内存和硬盘。对于日志分析类查询并发要求较低的场景，可以考虑采用机械硬盘作为存储，同时按照1:50的比例配置内存和硬盘。单节点存储数据建议在2TB以内，最大不要超过5TB，避免查询速度慢、系统不稳定。



## 内存建议

给JVM配置机器一半的内存，但是不建议超过32G

修改 conf/jvm.options 配置，-Xms和-Xmx设置为相同的值，推荐设置为机器内存的一半左右，剩余一半留给操作系统缓存使用。jvm内存建议不要低于2G，否则有可能因为内存不足导致ES无法正常启动或内存溢出，jvm建议不要超过32G，否则jvm会禁用内存对象指针压缩技术，造成内存浪费。机器内存大于64G内存时，推荐配置-Xms30g -Xmx30g 。



## 主节点建议

规模较大的集群配置专有主节点，避免脑裂问题。

Elasticsearch主节点（master节点）负责集群元信息管理、index的增删操作、节点的加入剔除，定期将最新的集群状态广播至各个节点。在集群规模较大时，建议配置专有主节点只负责集群管理，不存储数据，不承担数据读写压力。

```properties
# 专有主节点配置(conf/elasticsearch.yml)：
node.master:true
node.data: false
node.ingest:false

# 数据节点配置(conf/elasticsearch.yml)：
node.master:false
node.data:true
node.ingest:true
```

Elasticsearch默认每个节点既是候选主节点，又是数据节点。最小主节点数量参数minimum_master_nodes推荐配置为候选主节点数量一半以上，该配置告诉Elasticsearch当没有足够的master候选节点的时候，不进行master节点选举，等master节点足够了才进行选举。

例如对于3节点集群，最小主节点数量从默认值1改为2。

```
# 最小主节点数量配置(conf/elasticsearch.yml)：
discovery.zen.minimum_master_nodes: 2
```



## 操作系统调优

````bash
# 关闭交换分区，防止内存置换降低性能。 将/etc/fstab 文件中包含swap的行注释掉
sed -i '/swap/s/^/#/' /etc/fstab
swapoff -a
# 单用户可以打开的最大文件数量，可以设置为官方推荐的65536或更大些
echo "* - nofile 655360" >> /etc/security/limits.conf
# 单用户线程数调大
echo "* - nproc 131072" >> /etc/security/limits.conf
# 单进程可以使用的最大map内存区域数量
echo "vm.max_map_count = 655360" >> /etc/sysctl.conf
````

参数修改立即生效：

```
sysctl -p
```



## 索引建议

设置合理的索引分片数和副本数。

索引分片数建议设置为集群节点的整数倍，初始数据导入时副本数设置为0，生产环境副本数建议设置为1（设置1个副本，集群任意1个节点宕机数据不会丢失；设置更多副本会占用更多存储空间，操作系统缓存命中率会下降，检索性能不一定提升）。单节点索引分片数建议不要超过3个，每个索引分片推荐10-40GB大小。索引分片数设置后不可以修改，副本数设置后可以修改。Elasticsearch6.X及之前的版本默认索引分片数为5、副本数为1，从Elasticsearch7.0开始调整为默认索引分片数为1、副本数为1。



## 查询建议

默认情况下，Elasticsearch的查询会计算返回的每条数据与查询语句的相关度，但对于非全文索引的使用场景，用户并不关心查询结果与查询条件的相关度，只是想精确的查找目标数据。此时，可以通过filter来让Elasticsearch不计算评分，并且尽可能的缓存filter的结果集，供后续包含相同filter的查询使用，提高查询效率。







