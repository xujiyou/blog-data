# Promethues 储存

官方文档地址：https://prometheus.io/docs/prometheus/latest/storage/

Prometheus 自身就是一个时间序列数据库，默认以文件方式储存，但也可以与远程储存系统集成。

## 本地存储

Prometheus 以自定义格式在磁盘上存储时间序列数据。

### 磁盘上的布局

抓取的数据分为两个小时。每个两个小时的时间段包含一个目录。该目录包含一个或多个块文件，该文件包含该时间窗口的所有时间序列样本，以及元数据文件和索引文件（该索引文件将度量标准名称和标签索引到块文件中的时间序列）。

通过API删除数据时，删除记录存储在单独的逻辑删除文件中（而不是立即从块文件中删除数据）。

当前传入样本的块保留在内存中，尚未完全保留。它通过预写日志（WAL）防止崩溃，当Prometheus服务器在崩溃后重新启动时可以重放该日志。预写日志文件`wal`以128MB的段存储在目录中。这些文件包含尚未压缩的原始数据，因此它们比常规的块文件大得多。Prometheus将至少保留3个预写日志文件，但是高流量服务器可能会看到三个以上的WAL文件，因为它需要保留至少两个小时的原始数据。

Prometheus服务器的数据目录的目录结构如下所示：

```
./data
├── 01BKGV7JBM69T2G1BGBGM6KB12
│   └── meta.json
├── 01BKGTZQ1SYQJTR4PB43C8PD98
│   ├── chunks
│   │   └── 000001
│   ├── tombstones
│   ├── index
│   └── meta.json
├── 01BKGTZQ1HHWHV8FBJXW1Y3W0K
│   └── meta.json
├── 01BKGV7JC0RY8A6MACW02A2PJD
│   ├── chunks
│   │   └── 000001
│   ├── tombstones
│   ├── index
│   └── meta.json
└── wal
    ├── 00000002
    └── checkpoint.000001
```

请注意，本地存储的局限性在于它不是集群或复制的。因此，面对磁盘或节点中断，它不是任意可伸缩的或持久的，应该像对待任何其他类型的单节点数据库一样对待它。建议将RAID用于磁盘可用性，[快照](https://prometheus.io/docs/prometheus/latest/querying/api/#snapshot)用于备份，容量计划等，以提高耐用性。通过适当的存储耐久性和计划，可以在本地存储中存储多年的数据。

或者，可以通过[远程读/写API](https://prometheus.io/docs/operating/integrations/#remote-endpoints-and-storage)使用外部存储。这些系统在耐用性，性能和效率上差异很大，因此需要仔细评估。

有关文件格式的更多详细信息，请参见[TSDB format](https://github.com/prometheus/prometheus/blob/master/tsdb/docs/format/README.md)。

## 压缩

最初的两个小时的块最终会在后台压缩为更长的块。

压缩将创建更大的块，最多保留时间的10％，即31天，以较小者为准。



## 管理

Prometheus 有几个配置本地存储的选项：

- --storage.tsdb.path`：这确定Prometheus在何处写入其数据库。默认为`data/
- `--storage.tsdb.retention.time`：这确定何时删除旧数据。默认为`15d`。
- `--storage.tsdb.retention.size`：[EXPERIMENTAL]这确定存储块可以使用的最大字节数（请注意，这不包括WAL大小，这可能是很大的）。最旧的数据将首先被删除。默认为`0`或禁用。该标志是实验性的，可以在将来的版本中进行更改。支持的单位：KB，MB，GB，PB。例如：“ 512MB”
- `--storage.tsdb.wal-compression`：此标志启用预写日志（WAL）的压缩。根据您的数据，可以预期WAL大小将减少一半，而额外的CPU负载却很少。请注意，如果启用此标志，然后将Prometheus降级到2.11.0以下的版本，则您将需要删除WAL，因为它将不可读。

平均而言，普罗米修斯每个样本仅使用大约1-2个字节。因此，要计划Prometheus服务器的容量，可以使用以下粗略公式：

```
needed_disk_space = retention_time_seconds * ingested_samples_per_second * bytes_per_sample
```

Prometheus 的本地存储并不意味着持久的长期存储。



## 远程存储集成

Prometheus的本地存储在可伸缩性和持久性方面受到单个节点的限制。Prometheus并没有尝试解决Prometheus本身中的群集存储，而是提供了一组允许与远程存储系统集成的接口。

### 总览

Prometheus通过两种方式与远程存储系统集成：

- Prometheus可以将提取的样本以标准格式写入远程URL。
- Prometheus可以以标准格式从远程URL读取（返回）样本数据。

读写协议都使用基于HTTP的快速压缩协议缓冲区编码。该协议尚未被认为是稳定的API，当可以安全地假定Prometheus和远程存储之间的所有跃点都支持HTTP / 2时，该协议将来可能会更改为在HTTP / 2上使用gRPC。







