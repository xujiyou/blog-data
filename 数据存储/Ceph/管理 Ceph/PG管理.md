# PG 管理

官方文档：https://ceph.readthedocs.io/en/latest/rados/operations/placement-groups/

PG 是 Ceph 中非常重要的概念。

## 自动扩展的 PG

系统中的每个池都有一个 `pg_autoscale_mode` 属性，可以将其设置为`off`，`on` 或 `warn`。

- `off`: 禁用此池的自动缩放。管理员可以为每个池选择适当的PG号。Please refer to [Choosing the number of Placement Groups](https://ceph.readthedocs.io/en/latest/rados/operations/placement-groups/#choosing-number-of-placement-groups) for more information.
- `on`: 启用给定池的PG计数的自动调整。
- `warn`: 当应调整PG数量时发出健康警报

要为现有池设置自动缩放模式，请执行以下操作：

```
ceph osd pool set <pool-name> pg_autoscale_mode <mode>
```

例如：

```bash
$ ceph osd pool set foo pg_autoscale_mode on
```

还可以使用以下命令配置适用于将来创建的任何池的默认pg_autoscale_mode：

```bash
$ ceph config set global osd_pool_default_pg_autoscale_mode <mode>
```



#### 查看状态

您可以使用以下命令查看每个池，池的相对利用率以及对PG计数的任何建议更改：

```bash
$ ceph osd pool autoscale-status
```



#### 自动缩放

最简单的方法是允许群集根据使用情况自动扩展PG。Ceph 将查看整个系统的可用存储总量和目标 PG 数量，查看每个池中存储了多少数据，并尝试相应地分配PG。该系统采用相对保守的方法，仅当当前PG数量（pg_num）超出其认为的3倍时才对池进行更改。

每个OSD的PG的目标数量基于可配置的mon_target_pg_per_osd（默认值：100），可以通过以下方式调整：

```bash
$ ceph config set global mon_target_pg_per_osd 100
```

自动缩放器将分析池并在每个子树的基础上进行调整。因为每个池可能映射到不同的CRUSH规则，并且每个规则可能在不同的设备之间分配数据，所以Ceph将考虑独立使用层次结构的每个子树。例如，映射到ssd类的OSD的池和映射到hdd类的OSD的池将分别具有最佳PG计数，这取决于这些相应设备类型的数量。



#### 指定预期的池大小

首次创建集群或池时，它将消耗集群总容量的一小部分，并且在系统中似乎只需要少量的放置组。但是，在大多数情况下，群集管理员会很好地了解哪些池将随着时间的推移消耗大部分系统容量。通过将此信息提供给Ceph，可以从一开始就使用更多数量的PG，从而避免pg_num的后续更改以及进行这些调整时与移动数据相关的开销。

池的目标大小可以通过两种方式指定：要么以池的绝对大小（即字节）为单位，要么以相对于设置了 target_size_ratio 的其他池的权重为单位。

比如：

```bash
$ ceph osd pool set mypool target_size_bytes 100T
```

会告诉系统mypool预计会占用100 TiB的空间。或者，：

```bash
$ ceph osd pool set mypool target_size_ratio 1.0
```

会告诉系统mypool预期相对于设置了target_size_ratio的其他池消耗1.0。如果mypool是群集中唯一的池，则意味着预期使用了总容量的100％。如果存在另一个具有target_size_ratio 1.0的池，则两个池都将使用50％的群集容量。

您还可以在创建时使用 `ceph osd pool create` 命令的可选 --target-size-bytes <bytes>或--target-size-ratio <ratio>参数设置池的目标大小。



#### 在池的PGS上指定界限

也可以为一个池指定最小数量的PG。这对于确定执行IO时客户端将看到的并行度数量的下限非常有用，即使池中大多数都是空的。设置下限可防止Ceph将PG编号减少（或建议减少）到配置的编号以下。

您可以使用以下方法设置池的最小PG数量：

```bash
$ ceph osd pool set <pool-name> pg_num_min <num>
```

您还可以使用ceph osd pool create命令的可选--pg-num-min <num>参数指定池创建时的最小PG计数。



## 预设 pg_num

在创建 pool 时制定 pg_num ：

```bash
$ ceph osd pool create {pool-name} [pg_num]
```

或者，可以显式提供pg_num。但是，是否指定pg_num值并不影响事实之后群集是否自动调整该值。要启用或禁用自动调整，请执行以下操作：

```bash
$ ceph osd pool set {pool-name} pg_autoscale_mode (on|off|warn)
```

传统上，每个OSD的PG的“经验法则”为100。加上平衡器（默认情况下也启用），每个OSD的50个PG的值可能比较合理。

挑战（自动缩放器通常会为您完成）是：

- 使每个池中的PG与池中的数据成比例
- 考虑到每个PG在OSD上的复制或擦除编码扇出之后，最终每个OSD会有50-100个PG



## PG 如何使用

展示位置组（PG）汇总了一个池中的对象，因为以每个对象为基础来跟踪对象的位置和对象元数据在计算上是昂贵的，即，具有数百万个对象的系统无法实际跟踪每个对象的位置。

Ceph客户端将计算对象应位于哪个放置组。它通过对对象ID进行哈希处理并基于已定义池中PG的数量和池ID来应用操作。有关详细信息，请参见将PG映射到OSD。

放置组中对象的内容存储在一组OSD中。例如，在大小为2的复制池中，每个放置组将在两个OSD上存储对象。

当放置组的数量增加时，将为新的放置组分配OSD。CRUSH函数的结果也将更改，并且先前放置组中的某些对象将被复制到新的放置组中，并从旧的放置组中删除。



## PG 权衡

数据的持久性以及所有OSD之间的均匀分配都需要更多的放置组，但应将其数量减少到最少，以节省CPU和内存。



#### 数据持久性

OSD发生故障后，数据丢失的风险会增加，直到完全恢复其中包含的数据为止。假设有一种情况会导致单个展示位置组中的数据永久丢失：

- OSD失败，并且它包含的对象的所有副本均丢失。对于放置组中的所有对象，副本的数量突然从三个减少到两个。
- Ceph通过选择一个新的OSD重新创建所有对象的第三个副本，开始对该放置组的恢复。
- 在同一放置组内的另一个OSD在新OSD完全填充第三个副本之前发生故障。然后，某些对象将只有一个幸存副本。
- Ceph选择了另一个OSD并保持复制对象以恢复所需的副本数。
- 在同一放置组中的第三个OSD在恢复完成之前发生故障。如果此OSD包含对象的唯一剩余副本，则它将永久丢失。













































