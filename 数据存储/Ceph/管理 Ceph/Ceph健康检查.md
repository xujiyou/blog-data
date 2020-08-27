# Ceph 健康检查

官方教程：https://ceph.readthedocs.io/en/latest/rados/operations/health-checks/

这一篇主要学习一下各个组件的状态。

查看集群状态：

```bash
$ ceph -s  #ceph状态是否正常，及配置运行状态
$ ceph -w  #实时查看数据写入情况
$ ceph health detail #如果集群有问题，会详细列出具体的pg或者osd
```



## Monitor 组件状态

mon 当前状态：

```bash
$ ceph mon stat
```

Mon 选举状态：

```bash
$ ceph quorum_status -f json-pretty
```

查看 mon 映射信息：

```bash
$ ceph mon dump
```

查看一个 mon 的详细状态：

```bash
$ sudo ceph daemon mon.test-kubenode-1  mon_status
```



Monitor 的状态列表如下：



#### MON_DOWN

当前有一个或多个 Mon 已关闭。该群集需要大多数（超过1/2）的 Mon 才能正常运行。当一个或多个 Mon 关闭时，客户端可能很难与群集建立初始连接，因为它们可能需要尝试更多地址才能到达运行中的 Mon。

通常，应尽快重新启动 Mon ，以减少子监视器失败导致服务中断的风险。



#### MON_CLOCK_SKEW

运行 ceph-mon 守护程序的主机上的时钟没有同步。如果群集检测到时钟偏差大于 `mon_clock_drift_allowed` ，则会发出此运行状况警报。

```bash
$ ceph config get mon mon_clock_drift_allowed
```

最好通过使用ntpd或chrony之类的工具同步时钟来解决。

如果使时钟保持紧密同步是不切实际的，还可以增加 `mon_clock_drift_allowed` 阈值，但是此值必须保持在 `mon_lease` 间隔以下，以确保 Mon 群集正常运行。

```bash
$ ceph config get mon mon_lease
```



#### MON_MSGR2_NOT_ENABLED

已启用ms_bind_msgr2选项，但未将一个或多个 Mon 配置为绑定到群集 monmap 中的 v2 端口。

这意味着特定于msgr2协议的功能（例如加密）在部分或全部连接上不可用。

在大多数情况下，可以通过发出以下命令来纠正此问题：

```bash
$ ceph mon enable-msgr2
```

该命令将更改为旧的默认端口 6789 配置的任何 Mon ，以继续侦听 `6789` 上的 v1 连接，并继续侦听新的默认 `3300` 端口上的 v2 连接。

如果将监视器配置为侦听非标准端口（不是6789）上的 v1 连接，则需要手动修改 monmap 。





#### MON_DISK_LOW

一台或多台 Mon 的磁盘空间不足。如果存储 Mon 数据库的文件系统上的可用空间（通常为 /var/ lib/ceph/mon）（以百分比表示）低于`mon_data_avail_warn`（默认值：30％），则触发此警报。

```bash
$ ceph config get mon mon_data_avail_warn
```

这可能表明系统上的某些其他进程或用户正在填充 Mon 使用的同一文件系统。它也可能表示 Mon 数据库很大（请参见下面的MON_DISK_BIG）。

如果无法释放空间，则可能需要将 Mon 的数据目录移动到另一个存储设备或文件系统（当然，在 Mon 未运行时）。



#### MON_DISK_CRIT

一台或多台 Mon 的磁盘空间严重不足。如果存储 Mon 数据库的文件系统上的可用空间（通常为 /var/lib/ceph/mon）（以百分比表示）低于`mon_data_avail_crit`（默认值：5％），则将触发此警报。参见上面的MON_DISK_LOW。

```bash
$ ceph config get mon mon_data_avail_crit
```



#### MON_DISK_BIG

一个或多个 Mon 的数据库大小非常大。如果监视器数据库的大小大于 `mon_data_size_warn`（默认值：15 GiB），则会触发此警报。

```bash
$ ceph config get mon mon_data_size_warn
```

大型数据库是不常见的，但不一定表示有问题。当某些 PG 长时间未达到 `active+clean` 状态时，Mon 数据库的大小可能会增加。

这也可能表明该 Mon 的数据库未正确压缩，这在某些较旧版本的 leveldb 和 rockdb 中已经观察到。使用`ceph daemon mon.<id> compact` 强制压缩可能会缩小磁盘上的大小。

此警告还可能指示 Mon 存在一个错误，该错误阻止 Mon 修剪其存储的群集元数据。如果问题仍然存在，请提 BUG。

可以通过以下方式调整警告阈值：

```bash
$ ceph config set global mon_data_size_warn <size>
```



## MGR 组件状态





























