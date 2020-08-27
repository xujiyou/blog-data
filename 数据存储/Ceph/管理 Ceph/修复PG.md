# 修复 PG

官方文档：https://ceph.readthedocs.io/en/latest/rados/operations/pg-repair/

有时，PG 可能会变得“不一致”。要将 PG 返回到 active+clean 状态，您必须首先确定哪个 PG 变得不一致，然后在其上运行`pg repair`命令。



## 定位问题

以下命令提供了ceph集群运行状况的高级（低细节）概述：

```bash
$ ceph health detail
```

以下命令提供了有关展示位置组状态的更多详细信息：

```bash
$ ceph pg dump --format=json-pretty
```

以下命令列出了不一致的 PG：

```bash
$ rados list-inconsistent-pg {pool}
```

以下命令列出了不一致的rados对象：

```bash
$ rados list-inconsistent-obj {pgid}
```

以下命令在给定的放置组中列出不一致的快照集：

```bash
$ rados list-inconsistent-snapset {pgid}
```



## 修复 PG

```bash
$ ceph pg repair {pgid}
```

“ pg repair”命令尝试修复各种不一致性。如果“ pg repair”发现不一致的放置组，它将尝试用权威副本的摘要覆盖不一致副本的摘要。如果“ pg repair”发现不一致的复制池，则将不一致的副本标记为丢失。对于复制池，恢复超出了“ pg修复”的范围。

对于擦除编码池和bluestore池，如果将osd_scrub_auto_repair（配置默认值为“ false”）设置为true，并且最多发现osd_scrub_auto_repair_num_errors（配置默认值为5）错误，则Ceph将自动修复。

“ pg修复”并不能解决所有问题。当发现位置不一致时，Ceph不会自动修复它们。

本段中的内容与文件存储有关，bluestore有其自己的内部校验和。匹配记录的校验和和计算得出的校验和不能证明权威副本实际上是权威的。在没有校验和可用的情况下，“ pg repair”将优先使用主数据库上的数据。这可能是也可能不是未损坏的副本。这就是为什么当发现不一致时需要人工干预的原因。人为干预有时意味着使用“ ceph-objectstore-tool”。



















