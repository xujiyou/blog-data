# 监控 Ceph 集群

官方文档：https://ceph.readthedocs.io/en/latest/rados/operations/monitoring/

拥有正在运行的群集后，可以使用 ceph 工具来监视群集。监视群集通常涉及检查 OSD 状态，Mon 状态，PG 状态和 MDS 状态。



## 命名行检查

要以交互方式运行ceph工具，请在命令行中不带任何参数键入ceph。例如：

```
$ ceph
ceph> health
ceph> status
ceph> quorum_status
ceph> mon stat
```

如果为配置或密钥环指定了非默认位置，则可以指定它们的位置：

```
ceph -c /path/to/conf -k /path/to/keyring health
```





## 检查集群健康状态

在启动集群之后，以及在开始读取和/或写入数据之前，请首先检查集群的状态。

```bash
$ ceph status
```

或：

```bash
$ ceph -s
```

或者在交互模式下：

```bash
ceph> status

```

结果如下：

```
  cluster:
    id:     150147a4-72f8-456d-8b8c-5204fb866be4
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum test-kubenode-1,test-kubenode-2,test-kubenode-3 (age 4h)
    mgr: test-kubenode-1(active, since 22h)
    osd: 16 osds: 16 up (since 4h), 16 in (since 4h)
 
  data:
    pools:   2 pools, 33 pgs
    objects: 15.52k objects, 50 GiB
    usage:   101 GiB used, 44 TiB / 44 TiB avail
    pgs:     33 active+clean
 
  io:
    client:   26 KiB/s rd, 1.2 MiB/s wr, 0 op/s rd, 23 op/s wr
```

> **Ceph 如何计算数据量**
>
> `usage` 反映了实际使用的原始存储量。xxx GB / xxx GB 表示群集的整体存储容量的可用数量。名义数字反映了复制，克隆或快照之前存储的数据的大小。因此，实际存储的数据量通常超过名义存储量，因为Ceph创建数据的副本，并且还可能使用存储容量进行克隆和快照。



## Watch

除了每个守护程序的本地日志记录之外，Ceph集群还维护一个集群日志，该日志记录有关整个系统的高级事件。这被记录到监视服务器上的磁盘（默认为/var/log/ceph/ceph.log），但也可以通过命令行进行监视。

要跟踪群集日志，请使用以下命令：

```bash
$ ceph -w
```

除了使用 `ceph -w` 打印发出的日志行之外，还可以使用 `ceph log last [n]` 查看集群日志中的最新n行。



## 网络性能检查

Ceph OSD 在它们之间发送心跳 ping 消息以监视守护程序的可用性。我们还使用响应时间来监视网络性能。虽然繁忙的 OSD 可能会延迟ping 响应，但是我们可以假设，如果网络交换机发生故障，则会在不同的 OSD 对之间检测到多个延迟。

默认情况下，会警告ping时间超过1秒（1000毫秒）。

```
HEALTH_WARN Slow OSD heartbeats on back (longest 1118.001ms)
```

运行状况详细信息将添加 OSD 的组合，这些组合将看到延迟以及延迟的程度。数量上限为10个。

要查看更多详细信息和完整的网络性能信息转储，可以使用dump_osd_network命令。通常，它会发送给 mgr ，但可以通过将其发布到任何OSD 来限制于特定 OSD 的交互。缺省值1秒（1000毫秒）的当前阈值可以作为参数（以毫秒为单位）被覆盖。

以下命令将通过指定阈值0并发送到mgr来显示所有收集的网络性能数据。

```bash
$ ceph daemon /var/run/ceph/ceph-mgr.test-kubenode-1.asok dump_osd_network 0
{
    "threshold": 0,
    "entries": [
        {
            "last update": "Wed Aug 26 15:14:53 2020",
            "stale": false,
            "from osd": 9,
            "to osd": 13,
            "interface": "front",
            "average": {
                "1min": 0.755,
                "5min": 0.631,
                "15min": 0.619
            },
            "min": {
                "1min": 0.529,
                "5min": 0.446,
                "15min": 0.442
            },
            "max": {
                "1min": 3.505,
                "5min": 3.505,
                "15min": 3.505
            },
            "last": 0.455
        },
    ]
}
```



## 静音

可以将运行状况检查静音，以使它们不会影响群集的总体报告状态。使用运行状况检查代码指定警报（请参阅运行状况检查）：

```bash
$ ceph health mute <code>
```

例如，如果存在运行状况警告，请将其静音将使群集报告HEALTH_OK的总体状态。例如，要使OSD_DOWN警报静音，请执行以下操作：

```bash
$ ceph health mute OSD_DOWN
```

静音被报告为ceph health命令的短格式和长格式的一部分。例如，在上述情况下，集群将报告：

```bash
$ ceph health
HEALTH_OK (muted: OSD_DOWN)
$ ceph health detail
HEALTH_OK (muted: OSD_DOWN)
(MUTED) OSD_DOWN 1 osds down
    osd.1 is down
```

可以使用以下方法明确删除静音：

```bash
$ ceph health unmute OSD_DOWN
```

设置静音时间：

```bash
$ ceph health mute OSD_DOWN 4h    # mute for 4 hours
$ ceph health mute MON_DOWN 15m   # mute for 15  minutes
```



## 统计信息

要检查集群在池中的数据使用情况和数据分布，可以使用df选项。它类似于Linux df。执行以下命令：

```bash
$ ceph df
--- RAW STORAGE ---
CLASS  SIZE    AVAIL   USED    RAW USED  %RAW USED
hdd    44 TiB  44 TiB  85 GiB   101 GiB       0.23
TOTAL  44 TiB  44 TiB  85 GiB   101 GiB       0.23
 
--- POOLS ---
POOL                   ID  STORED   OBJECTS  USED     %USED  MAX AVAIL
device_health_metrics   1  255 KiB       16  510 KiB      0     21 TiB
kubernetes              2   42 GiB   15.52k   84 GiB   0.20     21 TiB
```

挺容易理解的。



## OSD 状态

查看 OSD 状态：

```bash
$ ceph osd stat
```

或者：

```bash
$ ceph osd dump
```

或：

```bash
$ ceph osd tree
```



## MDS 状态

```bash
$ ceph mds stat
```

或：

```bash
$ ceph fs dump
```















