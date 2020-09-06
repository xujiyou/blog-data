# Prometheus 模块

官方文档：https://ceph.readthedocs.io/en/latest/mgr/prometheus/

提供了一个面向Prometheus 的指标导出路径，Ceph-mgr从所有MgrClient进程（例如，mons和OSD）接收具有性能计数器架构数据和实际计数器数据的MMgrReport消息，并保留最后N个样本的循环缓冲区。该模块创建一个HTTP端点（像所有Prometheus导出器一样），并在被轮询（或在Prometheus术语中“pull”）时检索每个计数器的最新样本。



## 开启

```bash
$ ceph mgr module enable prometheus
```



## 配置

> 需要重新启动Prometheus管理器模块才能应用配置更改。

Prometheus 模块默认使用 9283 端口，端口和监听地址都可以使用`ceph config-key set`设置进行配置，`mgr/prometheus/server_addr` 和 `mgr/prometheus/server_port` 为 ip 地址和端口：

```bash
$ ceph config set mgr mgr/prometheus/server_addr 0.0.0.0
$ ceph config set mgr mgr/prometheus/server_port 9283
```

>  应该始终将此模块的 scrape_interval 设置为与Prometheus的 pull 间隔匹配，以使其正常工作，并且不会引起任何问题。

默认情况下，Prometheus管理器模块配置的 pull 间隔为15秒。模块中的 pull 间隔用于缓存目的，并确定何时缓存过期。

不建议在10秒以下使用 pull 间隔。建议使用15秒作为 pull 间隔，但是在某些情况下，增加刮擦间隔可能会很有用。

要在Prometheus模块中设置不同的 pull 间隔，请将scrape_interval设置为所需的值：

```bash
$ ceph config set mgr mgr/prometheus/scrape_interval 20
```

在大型群集（> 1000个OSD）上，获取指标的时间可能变得很长，如果没有缓存，则Prometheus管理器模块（尤其是与多个Prometheus实例结合使用）可能会使管理器超载，并导致Ceph管理器实例无响应或崩溃。因此，缓存默认情况下处于启用状态，无法禁用。这意味着缓存有可能过时。当从Ceph获取指标的时间超过配置的scrape_interval时，缓存被认为是过时的。

在这种情况下，将记录警告，模块将：

- 返回 503
- 返回缓存的内容，即使它可能已过期。

可以配置此行为。默认情况下，它将返回503 HTTP状态代码（服务不可用）。您可以使用ceph config set命令设置其他选项。

要告诉模块以可能过时的数据响应，请将其设置为返回：

```bash
$ ceph config set mgr mgr/prometheus/stale_cache_strategy return
```

要告诉模块响应“服务不可用”，请将其设置为失败：

```bash
$ ceph config set mgr mgr/prometheus/stale_cache_strategy fail
```



## RBD IO统计

通过启用动态OSD性能计数器，该模块可以选择收集RBD每映像IO统计信息。收集在`mgr/prometheus/rbd_stats_pools`配置参数中指定的池中所有映像的统计信息。该参数是 `pool[/namespace]` 条目的逗号或空格分隔列表。如果未指定名称空间，则将收集池中所有名称空间的统计信息。

激活启用了RBD的池pool1，pool2和poolN的示例：

```bash
$ ceph config set mgr mgr/prometheus/rbd_stats_pools "pool1,pool2,poolN"
```

该模块列出扫描指定的池和名称空间的所有可用映像的列表，并定期刷新它。该周期可以通过 `mgr/prometheus/rbd_stats_pools_refresh_interval` 参数（以秒为单位）进行配置，默认为300秒（5分钟）。如果模块从先前未知的RBD映像中检测到统计信息，则将强制更早刷新。

将同步间隔设置为10分钟的示例：

```bash
$ ceph config set mgr mgr/prometheus/rbd_stats_pools_refresh_interval 600
```



## 统计 Name 和 lables

统计信息的名称与Ceph命名的名称完全相同，非法字符。，-和::转换为`_` 和`ceph_`。

所有守护程序统计信息都有一个ceph_daemon标签，例如“ osd.123”，用于标识它们来自的守护程序的类型和ID。一些统计信息可能来自不同类型的守护程序，因此在查询时OSD的RocksDB统计信息，您可能希望对以“ osd”开头的ceph_daemon进行过滤，以免混入监视器的rockdb统计信息。

集群统计信息（即Ceph集群全局的统计信息）具有适合其报告内容的标签。例如，与池有关的度量标准具有pool_id标签。

来自核心Ceph的直方图的长期平均值由一对<name> _sum和<name> _count度量表示。这类似于在Prometheus中表示直方图的方式，也可以类似地对其进行处理。



## Pool 和 OSD 元数据

输出特殊系列，以允许在某些元数据字段上显示和查询。

Pool 有一个 `ceph_pool_metadata` 字段，字段如下：

```
ceph_pool_metadata{pool_id="2",name="cephfs_metadata_a"} 1.0
```

OSD 有一个 `ceph_osd_metadata` 字段：

```
ceph_osd_metadata{cluster_addr="172.21.9.34:6802/19096",device_class="ssd",ceph_daemon="osd.0",public_addr="172.21.9.34:6801/19096",weight="1.0"} 1.0
```























