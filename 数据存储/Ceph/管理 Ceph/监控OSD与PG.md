# 监控 OSD 与 PG

官方文档：https://ceph.readthedocs.io/en/latest/rados/operations/monitoring-osd-pg/

高可用性和高可靠性要求使用容错方法来管理硬件和软件问题。 Ceph没有单一的故障点，并且可以以“降级”模式处理对数据的请求。

Ceph引入了一个名为 PG 的间接层，以确保数据不直接绑定到特定的OSD地址，这意味着要跟踪系统故障，需要找到问题根源的 PG 和底层OSD。

> 集群某一部分的故障可能会阻止您访问特定对象，但这并不意味着您无法访问其他对象。当您遇到故障时，请不要惊慌。只需按照监视OSD和放置组的步骤进行即可。然后，开始故障排除。

Ceph通常是自我修复。但是，如果问题仍然存在，则监视OSD和 PG 将有助于您确定问题。



## 监控 OSD

OSD 的状态是在群集中（in）或不在群集中（out）；它可以是正在运行（up），或者已关闭（down）。如果 OSD 已启动，则它可能位于群集中（可以读取和写入数据），也可能不在群集中。如果它在集群中并且最近移出了集群，则Ceph会将 PG 迁移到其他OSD。如果OSD不在群集中，则 CRUSH 不会将 PG 分配给 OSD 。如果OSD关闭，则也应该是 out 状态。

> 如果OSD处于 down 和 in 状态，则出现了问题，群集将无法处于正常状态。

![../../../_images/174325a044ac2a293e36baa0b41fcd253528f7e6f542c0e68a31da37f968cdc2.png](../../../resource/174325a044ac2a293e36baa0b41fcd253528f7e6f542c0e68a31da37f968cdc2.png)

如果执行诸如ceph health，ceph -s或ceph -w之类的命令，您可能会注意到集群并不总是回显HEALTH OK。不要惊慌对于OSD，在某些预期情况下，您应该期望集群不会回显HEALTH OK：

1. 您尚未启动集群（它不会响应）。
2. 您刚刚启动或重新启动了集群，但集群尚未准备就绪，因为正在创建展示位置组并且OSD正在对等。
3. You just added or removed an OSD.
4. You just have modified your cluster map.

监视OSD的一个重要方面是要确保在群集启动和运行时，群集中的所有OSD也在启动和运行。要查看所有OSD是否正在运行，请执行：

```bash
$ ceph osd stat
16 osds: 16 up (since 5h), 16 in (since 5h); epoch: e84022
```

是 map 的版本数。

如果集群中的OSD数量大于已启动的OSD数量，请执行以下命令以标识未运行的ceph-osd守护程序：

```bash
$ ceph osd tree
```

> 通过精心设计的 CRUSH 层次结构进行搜索的能力可以通过更快地识别物理位置来帮助您对群集进行故障排除。

如果 OSD down 了，启动它：

```bash
$ sudo systemctl start ceph-osd@1
```

更多 OSD 故障请查看：https://ceph.readthedocs.io/en/latest/rados/troubleshooting/troubleshooting-osd/#osd-not-running



## PG 集合

当 CRUSH 将 PG 分配给OSD时，它查看 Pool 的副本数，并将 PG 分配给OSD，以便将 PG 的每个副本分配给不同的OSD。例如，如果 Pool 需要一个 PG 的三个副本，则 CRUSH 可以将它们分别分配给osd.1，osd.2和osd.3。

实际上，CRUSH 会寻找一个伪随机放置，该放置将考虑在 CRUSH 映射中设置的故障域，因此您很少会看到分配给大型群集中最近邻居OSD的 PG。我们将应包含特定 PG 副本的 OSD 称为 Acting Set。在某些情况下，代理集中的OSD处于关闭状态，或者无法为 PG 中的对象提供服务。当出现这些情况时，不要惊慌。常见的示例包括：

- 您添加或删除了OSD。然后，CRUSH将布局组重新分配给其他OSD，从而更改了“Acting Set”的组成并通过“回填”过程生成了数据迁移。
- OSD down，重新启动，会处于 recovering 状态。
- 代理集中的一个OSD已关闭或无法满足请求，另一个 OSD 会暂时承担其职责。

Ceph使用 Up Set 处理客户端请求，Up Set 是实际上将处理请求的OSD集合。在大多数情况下，Up Set和Acting Set实际上是相同的。如果不是，则可能表明 Ceph 正在迁移数据，正在恢复 OSD 或存在问题（即在这种情况下Ceph通常以“stuck stale”状态显示）。

要检索展示 PG 列表，请执行以下操作：

```bash
$ ceph pg dump
```

要查看给定展示位置组的“Acting Set”或“Up Set ”中的哪些OSD，请执行以下操作：

```
ceph pg map {pg-num}
```

例如：

```bash
$ ceph pg map 2.e
osdmap e84036 pg 2.e (2.e) -> up [2,9] acting [2,9]
```

> 如果Up Set和Acting Set不匹配，则可能表明群集重新平衡自身或群集存在潜在问题。



## PEERING

在将数据写入展示 PG 之前，它必须处于 `avtive` 状态，并且应处于 `clean` 状态。为了让 Ceph 确定 PG 的当前状态，该 PG 的主OSD（即Acting Set中的第一个OSD），与二级和三级OSD进行对话，以就 PG 的当前状态建立协议（假设有PG的三个副本的池）

![../../../_images/02f607b45842e47da91dce4eb18e312c2560d41d5bad3ca34ff62779ac5e00a0.png](../../../resource/02f607b45842e47da91dce4eb18e312c2560d41d5bad3ca34ff62779ac5e00a0.png)

OSD还向监视器报告其状态。



## 监控 PG 状态

查看 PG 状态：

```bash
$ ceph pg stat
33 pgs: 33 active+clean; 50 GiB data, 85 GiB used, 44 TiB / 44 TiB avail; 597 KiB/s wr, 21 op/s
```

> PG ID
>
> PG ID由 pool id（不是 pool 名称），后跟句点（.）和 PG ID（十六进制数）组成。可以从 `ceph osd lspools`的输出中查看 Pool ID及其名称。例如，创建的第一个池 Pool 对应于池号1。完全合格的展示位置组ID的格式如下：
>
> ```
> {pool-num}.{pg-id}
> ```
>
> 一个 PG ID：
>
> ```
> 1.1f
> ```

要检索 PG 列表，请执行以下操作:

```bash
$ ceph pg dump
```

输出 json 文件：

```bash
$ ceph pg dump -o {filename} --format=json
```

要查询特定的 PG ，请执行以下操作：

```bash
$ ceph pg {poolnum}.{pg-id} query
```

PG 的状态如下：



#### CREATING

创建 Pool 时，它将创建您指定的 PG 数。当 Ceph 创建一个或多个 PG 时，它将显示 `creating`。创建 OSD 之后，属于 PG “Acting Set”的OSD 将会对等。对等完成后，PG 状态应为`active+clean`，这意味着 Ceph 客户端可以开始写入展示位置组。

![../../../_images/f5792aa71980fbe30166d3a5cd03af0c205e3befdae1f19636c3b848da2e5418.png](../../../resource/f5792aa71980fbe30166d3a5cd03af0c205e3befdae1f19636c3b848da2e5418.png)



#### PEERING

当Ceph对等 PG 时，Ceph 会将存储该 PG 副本的 OSD 置于关于该 PG 中对象和元数据状态的协议中。当 Ceph 完成对等时，这意味着存储PG 的 OSD 同意 PG 的当前状态。但是，对等过程的完成并不意味着每个副本都具有最新的内容。

> Ceph不会确认对客户端的写操作，直到行为集的所有OSD都坚持写操作为止。这种做法可确保至少从上一次成功的对等操作以来，每个动作集的成员都有记录。
>
> 通过准确记录每个已确认的写入操作，Ceph可以构建和传播 PG 的完整新权威历史记录，以及一套完整有序的操作（如果执行的话），将使OSD的展示位置组副本保持最新状态。

#### ACTIVE

Ceph完成对等过程后，PG 可能会变为 `active` 状态。active 状态表示 PG 中的数据通常在主 PG 和副本中可用于读取和写入操作。



#### CLEAN

当 PG 处于 `clean` 状态时，主 OSD 和副本 OSD 已成功对等，并且该 PG 没有杂散副本。 Ceph 将展示位置组中的所有对象复制了正确的次数。



#### DEGRADED

当客户端将对象写入主OSD时，主OSD负责将副本写入副本OSD。在主OSD将对象写入存储后，放置组将保持 `degraded` 状态，直到主OSD从副本 OSD 收到 Ceph 成功创建副本对象的确认为止。

PG 可以处于`active+degraded`的原因是，即使 OSD 尚未容纳所有对象，也可能处于活动状态。如果 OSD 出现故障，Ceph会将分配给 OSD的每个 PG 标记为 `degraded`。当 OSD 重新联机时，OSD 必须再次对等。但是，客户端仍然可以将新对象写入 `degraded` 的 PG（如果处于 `active` 状态）。

如果 OSD down 并且 `degraded` 的情况持续存在，则 Ceph 可能会将关闭的 OSD 标记为不在群集中，并将数据从关闭的 OSD 重新映射到另一个 OSD。标记到标记之间的时间由 `mon osd down out interval` 控制，默认情况下，该间隔设置为600秒。

PG 也可能 `degraded` ，因为 Ceph 找不到 Ceph 认为应该在该 PG 中的一个或多个对象。尽管无法读取或写入未找到的对象，但仍可以访问`degraded`的放置组中的所有其他对象。



#### RECOVERING

Ceph 被设计用于在硬件和软件问题持续存在的规模上的容错能力。当 OSD 发生故障的一段时间内，OSD可能出现 `recovering` 状态。

恢复并不总是那么简单，因为硬件故障可能会导致多个OSD的级联故障。例如，机架或机柜的网络交换机可能发生故障。解决故障后，每个OSD都必须恢复。

Ceph 提供了许多设置来平衡新服务请求之间的资源争用与恢复数据对象并将 PG 还原到当前状态的需求。`osd recovery delay start` 设置允许 OSD 在开始恢复过程之前重新启动，重新对等甚至处理一些重播请求。`osd recovery thread timeout` 设置线程超时，因为多个 OSD 可能会失败，请以交错速率重新启动并重新对等。`osd recovery max active` 设置限制 OSD 将同时接受的恢复请求的数量，以防止OSD无法服务。`osd recovery max chunk` 设置限制了恢复的数据块的大小，以防止网络拥塞。



#### BACK FILLING

当新的OSD加入群集时，CRUSH将把放置组从群集中的OSD重新分配给新添加的OSD。强制新OSD立即接受重新分配的放置组会给新OSD带来过多的负担。用放置组回填OSD可使此过程在后台开始。回填完成后，新的OSD将在准备就绪后开始处理请求。

在回填操作期间，您可能会看到以下几种状态之一： `backfill_wait`表示回填操作处于待处理状态，但尚未进行；`backfilling`表示正在进行回填操作；并且`backfill_toofull`表示已请求回填操作，但由于存储容量不足而无法完成。当无法重新填充展示位置组时，可以考虑使用`incomplete`。

该`backfill_toofull`状态可以是瞬态的。随着PG的移动，空间可能变得可用。在`backfill_toofull`类似于`backfill_wait`只要条件改变回填可以继续在这。



#### REMAPPED

当为 PG 提供服务的 Acting Set 发生更改时，数据将从旧的 Acting Set 迁移到新的代理集。新的主 OSD 可能需要一些时间来处理请求。因此，它可能会要求旧的主要数据库继续为请求提供服务，直到完成 PG 迁移为止。数据迁移完成后，映射将使用新行为集的主OSD。



#### STALE

当Ceph使用心跳确保主机和守护程序正在运行时，这些 `ceph-osd`守护程序也可能进入一种`stuck`状态，即它们没有及时报告统计信息（例如，临时网络故障）。默认情况下，OSD守护程序每半秒（即`0.5`）报告启动，失败和启动统计信息的放置组，其频率比心跳阈值高。如果放置组的操作集中的**主OSD**无法向监视器报告，或者其他OSD已报告了主OSD `down`，则监视器将标记该放置组`stale`。

启动集群时，通常会看到`stale`状态，直到对等过程完成。在集群运行了一段时间之后，看到处于该`stale`状态的放置组，则表明这些放置组的主OSD是否正在`down`向监视器报告放置组统计信息。



## 识别故障 PG

如前所述，放置组不一定仅由于其状态不是 active+clean 而存在问题。通常，当 PG 卡住时，Ceph的自我修复功能可能会失效。卡住的状态包括：

- **Unclean**: PG 包含未复制所需次数的对象。他们应该正在恢复。
- **Inactive**: PG 无法处理读取或写入，因为它们正在等待 OSD 获取最新数据。
- **Stale**:  PG 处于未知状态，因为承载它们的 OSD 已有一段时间未报告给监视器集群(configured by `mon osd report timeout`).

要确定卡住的放置组，请执行以下操作：

```bash
$ ceph pg dump_stuck [unclean|inactive|stale|undersized|degraded]
```



## 查看对象的位置

要将对象数据存储在 Ceph 对象存储中，Ceph 客户端必须：

1. Set an object name
2. Specify a [pool](https://ceph.readthedocs.io/en/latest/rados/operations/pools)

Ceph 客户端检索最新的群集 Map，CRUSH 算法计算如何将对象映射到 PG ，然后计算如何将 PG 动态分配给OSD。

要找到对象位置，您只需要对象名称和池名称即可。例如：

```bash
$ ceph osd map {poolname} {object-name} [namespace]
```

> 练习：找到一个对象
>
> 作为练习，让我们创建一个对象。在命令行上使用rados put命令指定对象名称，包含一些对象数据的测试文件的路径以及池名称。例如：
>
> ```bash
> $ rados put {object-name} {file-path} --pool=data
> $ rados put test-object-1 testfile.txt --pool=data
> ```
>
> 要验证Ceph对象存储中是否存储了对象，请执行以下操作：
>
> ```bash
> $ rados -p data ls
> ```
>
> 现在，确定对象位置：
>
> ```bash xxzzxxzxz
> $ ceph osd map {pool-name} {object-name}
> $ ceph osd map data test-object-1
> ```
>
> Ceph应该输出对象的位置。例如：
>
> ```
> osdmap e537 pool 'data' (1) object 'test-object-1' -> pg 1.d1743484 (1.4) -> up ([0,1], p0) acting ([0,1], p0)
> ```
>
> 要删除测试对象，只需使用rados rm命令将其删除。例如：
>
> ```bash
> $ rados rm test-object-1 --pool=data
> ```
>
> 































