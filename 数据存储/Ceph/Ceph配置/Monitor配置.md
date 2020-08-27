# Monitor 配置

Monitor 是 Ceph 中最重要的组件，相应的，配置就会多。

官方文档：https://ceph.readthedocs.io/en/latest/rados/configuration/mon-config-ref/

了解如何配置 Ceph Monitor 是构建可靠的 Ceph 存储群集的重要部分。所有Ceph存储群集均至少具有一个 Monitor 节点。监视器配置通常会保持一致，也可以在群集中添加，删除或替换 Monitor。



## 背景知识

Ceph Monitors 维护集群 map 的“主副本”，这意味着 Ceph 客户端只需连接到一个 Ceph Monitor 并检索当前的集群 Map，就可以确定所有Ceph Monitor、Ceph OSD 守护程序和 Ceph 元数据服务器的位置。

在 Ceph 客户端可以读取或写入 Ceph OSD 守护程序或 Ceph 元数据服务器之前，它们必须首先连接到 Ceph 监视器。

有了集群 Map 的当前副本和 CRUSH 算法，Ceph 客户端可以计算任何对象的位置。计算对象位置的能力使 Ceph 客户端可以直接与 Ceph OSD 守护进程通信，这是 Ceph 的高可伸缩性和高性能的一个非常重要的方面。



Ceph Monitor 的主要作用是维护集群 Map 的主副本。 Ceph Monitors 还提供身份验证和日志服务。Ceph Monitors 将 Monitor 服务中的所有更改写入单个 Paxos 实例，并且 Paxos 将更改写入键/值存储，以实现强一致性。在同步操作期间，Ceph Monitors 可以查询集群映射的最新版本。Ceph Monitors 利用键/值存储的快照和迭代器（使用 leveldb ）执行存储范围的同步。



## 集群 Map

集群 Map 是各种 Map 的组合，包括Monitor Map，OSD Map，PG Map和 MDS Map。集群 Map 跟踪了许多重要的事情：哪些进程在Ceph存储集群中；集群中的哪些进程已启动，正在运行或已关闭；展示 PG 是处于活动状态还是非活动状态，并且处于 clean 状态或处于其他某种状态；以及反映群集当前状态的其他详细信息，例如存储空间的总量和使用的存储量。

当集群状态发生重大变化时（例如，Ceph OSD守护进程关闭，PG 进入降级状态等），集群 Map 将更新以反映集群的当前状态。此外，Ceph Monitor 还维护集群先前状态的历史记录。Monitor Map，OSD Map，PG Map和 MDS Map 分别维护其 Map 版本的历史记录。我们称每个版本为 `epoch`。

在操作 Ceph Storage Cluste r时，跟踪这些状态是系统管理职责的重要组成部分。



## Monitor 选主

群集可以在单个监视器上正常运行；但是，单个监视器是单个故障点。为了确保生产 Ceph 存储群集中的高可用性，应该使用多个 Moonitor 运行 Ceph ，以便单个 Monitor 的故障不会导致整个群集崩溃。

当一个Ceph Storage Cluster运行多个Ceph Monitor以获得高可用性时，Ceph Monitors 使用 Paxos 建立有关主群集映射的共识。达成共识需要大多数监视器运行以建立仲裁集群共识的法定人数（例如1； 3 中的 2 个； 5 中的 3 个； 6 中的 4 个；等等）。



`mon force quorum join`

- Description：即使先前已将其从地图中删除，也强制监视器加入仲裁
- Type：Boolean
- Default：`False`



## 一致性

在将 Monitor 配置添加到 Ceph 配置文件时，需要了解 Ceph Monitors 的某些体系结构方面的知识。当在集群中发现另一个 Ceph Monitor 时，Ceph 对 Ceph Monitor 施加强一致性要求。而 Ceph 客户端和其他 Ceph 守护程序使用 Ceph 配置文件来发现 Monitor ，Monitor 使用 Monitor 映射（monmap）而不是 Ceph 配置文件来发现彼此。

当在 Ceph 存储群集中发现其他 Ceph Monitor 时，Ceph Monitor 始终引用 monmap 的本地副本。使用 monmap 而不是 Ceph 配置文件可避免可能会破坏群集的错误（例如，在指定 Monitor 地址或端口时，ceph.conf 中的错别字）。由于 Monitor 使用 monmap 进行发现，并且它们与客户端和其他 Ceph 守护程序共享 monmap ，因此 monmap 为监视器提供了严格的保证，即它们的共识是有效的。

强一致性也适用于对monmap的更新。与Ceph Monitor上的任何其他更新一样，对 monmap 的更改始终通过称为 Paxos 的分布式共识算法运行。Ceph Monitor 必须同意对 monmap 的每次更新，例如添加或删除 Ceph Monitor，以确保仲裁中的每个 Monitor 都具有相同版本的monmap。对 monmap 的更新是增量更新，因此 Ceph Monitors 具有最新的议定版本和一组先前版本。维护历史记录可使具有旧版 monmap 的 Ceph Monitor 赶上 Ceph 存储集群的当前状态。

如果 Ceph Monitors 通过 Ceph 配置文件而不是通过 monmap 彼此发现，则将带来其他风险，因为Ceph配置文件不会自动更新和分发。Ceph 监视器可能会无意中使用较旧的 Ceph 配置文件，无法识别 Ceph 监视器，无法达到法定人数或出现 Paxos 无法准确确定系统当前状态的情况。





## 启动 Monitor

在大多数配置和部署情况下，部署Ceph的工具可能会通过为您生成 Monitor Map（例如ceph-deploy等）来帮助引导Ceph监视器。Ceph监视器需要一些明确的设置：

- **Filesystem ID**：fsid 是对象存储库的唯一标识符。由于可以在同一硬件上运行多个集群，因此在引导监视器时必须指定对象存储的唯一ID。部署工具通常会为您执行此操作（例如，ceph-deploy 可以调用 uuidgen 之类的工具），但是您也可以手动指定 fsid 。
- **Monitor ID**：Monitor ID 是分配给集群中每个监视器的唯一 ID 。它是一个字母数字值，按照惯例，标识符通常遵循字母增量（例如a，b等）。可以通过部署工具或使用 ceph 命令行在 Ceph 配置文件（例如[mon.a]，[mon.b]等）中进行设置。
- **Keys**：Monitor 必须具有密钥。部署工具（例如ceph-deploy）通常会为您执行此操作，但是您也可以手动执行此步骤。



`fsid`

- Description：集群 ID，每个集群都是唯一的
- Type：UUID
- Required：Yes.
- Default：N/A. 如果未指定，则可以由部署工具生成。

## 配置 Monitor

要将配置设置应用于整个集群，请在[global]下输入配置设置。要将配置设置应用于群集中的所有监视器，请在[mon]下输入配置设置。要将配置设置应用于特定的监视器，请指定监视器实例（例如[mon.a]）。按照约定，监视器实例名称使用字母表示法：

```ini
[global]

[mon]

[mon.a]

[mon.b]

[mon.c]
```



#### 最小配置

通过 Ceph 配置文件为 Ceph Monitor 设置的最低 Monitor 设置包括每个 Monitor 的主机名和 Monitor 地址。您可以在[mon]或特定 Monitor 的条目下进行配置。 

```ini
[global]
        mon host = 10.0.0.2,10.0.0.3,10.0.0.4
```

```ini
[mon.a]
        host = hostname1
        mon addr = 10.0.0.10:6789
```

一旦部署了Ceph集群，就不应更改 Monitor 的IP地址。但是，如果您决定更改 Monitor 的IP地址，则必须遵循特定的步骤。

修改参考：https://ceph.readthedocs.io/en/latest/rados/operations/add-or-rm-mons/#changing-a-monitor-s-ip-address



#### 初始成员

建议使用至少三个Ceph Monitor运行生产的Ceph Storage Cluster，以确保高可用性。当运行多个 Monitor 时，可以指定初始 Monitor ，这些 Monitor 必须是集群的成员才能建立仲裁。这可以减少群集联机所需的时间。

```ini
[mon]     
     		mon initial members = a,b,c
```

`mon initial members`

- Description：启动期间群集中初始 Monitor的ID。如果指定，则Ceph要求使用奇数个 Monitor 来形成初始仲裁（例如3个）。
- Type：String
- Default：None

> 群集中的大多数 Monitor 必须能够相互连接才能建立仲裁。您可以减少监视器的初始数量，以使用此设置建立仲裁。



#### 数据

Ceph提供了Ceph Monitors存储数据的默认路径。为了在生产 Ceph 存储集群中获得最佳性能，我们建议在Ceph OSD守护程序的单独主机和驱动器上运行Ceph Monitor。由于 leveldb 使用 mmap() 写入数据，因此 Ceph Monitors 经常将其数据从内存中刷新到磁盘上，如果数据存储与OSD 守护程序位于同一位置，则这可能会干扰 Ceph OSD 守护程序的工作负载。

Ceph Monitor 将其数据存储为键/值对。 Ceph Monitor需要 ACID 事务。使用数据存储区可防止通过 Paxos 恢复 Ceph Monitor运行损坏的版本，另外，它还可以在一个原子批处理中进行多个修改操作。

通常，不建议更改默认数据位置。如果修改默认位置，建议通过在配置文件的[mon]部分中进行设置，使其在Ceph Monitors中保持一致。



`mon data`

- Description：Monitor 的数据储存位置
- Type：String
- Default：`/var/lib/ceph/mon/$cluster-$id`

`mon data size warn`

- Description：当监视器的数据存储空间超过 15GB 时，在群集日志中发出 HEALTH_WARN 告警。 
- Type：Integer
- Default：`15*1024*1024*1024`

`mon data avail warn`

- Description：当监视器的数据存储的可用磁盘空间小于或等于此百分比时，在群集日志中发出 HEALTH_WARN 告警。
- Type：Integer
- Default：`30`

`mon data avail crit`

- Description：当监视器的数据存储的可用磁盘空间小于或等于此百分比时，在群集日志中发出HEALTH_ERR。
- Type：Integer
- Default：`5`

`mon warn on cache pools without hit sets`

- Description

  如果缓存池未配置 hit_set_type 值，请在群集日志中发出 HEALTH_WARN 。有关更多详细信息，请参见[hit_set_type](https://ceph.readthedocs.io/en/latest/rados/operations/pools/#hit-set-type)。

- Type：Boolean

- Default：`True`

`mon warn on crush straw calc version zero`

- Description

  如果 CRUSH 的 straw_calc_version 为零，则在群集日志中发出HEALTH_WARN。有关详细信息，请参见[CRUSH map tunables](https://ceph.readthedocs.io/en/latest/rados/operations/crush-map/#crush-map-tunables)。

- Type：Boolean

- Default：`True`

`mon warn on legacy crush tunables`

- Description：如果CRUSH可调参数太旧（早于mon_min_crush_required_version），请在集群日志中发出HEALTH_WARN
- Type：Boolean
- Default：`True`

`mon crush min required version`

- Description：集群所需的最低可调配置文件版本. See [CRUSH map tunables](https://ceph.readthedocs.io/en/latest/rados/operations/crush-map/#crush-map-tunables) for details.
- Type：String
- Default：`hammer`

`mon warn on osd down out interval zero`

- Description

  如果 mon osd down out 间隔为零，请在群集日志中发出HEALTH_WARN。在 leader 上将此选项设置为零的行为很像 `noout` 标志。 如果没有设置 noout 标志，但行为却完全相同，则很难弄清楚集群出了什么问题，因此我们在这种情况下报告警告。

- Type：Boolean

- Default：`True`

`mon warn on slow ping ratio`

- Description

  如果OSD之间的任何心跳超过了 `osd heartbeat grace` 的 `mon warn on slow ping ratio` 的警告，请在群集日志中发出HEALTH_WARN。默认值为5％。

- Type：Float

- Default：`0.05`

`mon warn on slow ping time`

- Description

  通过一个特殊的值覆盖 `mon warn on slow ping ratio` 。如果 OSD 之间的 ping 时间超过  `mon warn on slow ping time`，在集群日志中发出 `HEALTH_WARN` 。The default is 0 (disabled).

- Type：Integer

- Default：`0`

`mon warn on pool no redundancy`

- Description：如果没有配置任何副本池，则在群集日志中发出HEALTH_WARN。
- Type：Boolean
- Default：`True`

`mon cache target full warn ratio`

- Description

  Position between pool’s `cache_target_full` and `target_max_object` where we start warning

- Type：Float

- Default：`0.66`

`mon health to clog`

- Description：启用定期向集群日志发送运行状况摘要。
- Type：Boolean
- Default：`True`

`mon health to clog tick interval`

- Description

  监控器将运行状况摘要发送到群集日志的频率（以秒为单位）（非正数会将其禁用）。如果当前运行状况摘要为空或与上次相同，则监控器不会将其发送到集群日志。

- Type：Float

- Default：`60.0`

`mon health to clog interval`

- Description

  监控器将运行状况摘要发送到群集日志的频率（以秒为单位）（非正数会将其禁用）。无论摘要是否更改，Monitor都会始终将摘要发送到群集日志。

- Type：Integer

- Default：`3600`



























