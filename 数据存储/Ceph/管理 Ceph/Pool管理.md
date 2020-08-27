# Pool 管理

官方文档：https://ceph.readthedocs.io/en/latest/rados/operations/pools/

Pool 是用于存储对象的逻辑分区。

当首次部署群集而不创建 Pool 时，Ceph使用默认 Pool 来存储数据。Pool 为提供：

- **Resilience**: 您可以设置允许多少OSD发生故障而不丢失数据。典型的配置存储一个对象和一个附加副本（即size = 2），可以确定副本的数量。对于纠删码池，它是编码块的数量（即纠删码配置文件中的m = 2）
- **Placement Groups**: 您可以设置池的 PG 的数量。一个典型的配置每个OSD使用大约100个 PG ，以提供最佳的平衡，而不会消耗太多的计算资源。设置多个池时，请确保为池和整个群集设置合理数量的 PG 。
- **CRUSH Rules**: 当您将数据存储在池中时，对象及其副本（或用于擦除编码池的块）在群集中的位置由 CRUSH 规则控制。如果默认规则不适用于您的用例，则可以为您的池创建自定义的CRUSH规则。
- **Snapshots**: 可以通过 `ceph osd pool mksnap` 创建快找, 您可以有效地为特定池拍摄快照。

要将数据组织到池中，可以列出，创建和删除池。您还可以查看每个池的利用率统计信息。



## 列出 Pool

```bash
$ ceph osd lspools
```



## 创建 Pool

Pool 中默认的 PG 数量配置如下：

```
osd pool default pg num = 100
osd pool default pgp num = 100
```

要创建 Pool，需执行：

```bash
$ ceph osd pool create {pool-name} [{pg-num} [{pgp-num}]] [replicated] \
     [crush-rule-name] [expected-num-objects]
$ ceph osd pool create {pool-name} [{pg-num} [{pgp-num}]]   erasure \
     [erasure-code-profile] [crush-rule-name] [expected_num_objects] [--autoscale-mode=<on,off,warn>]
```

`{pool-name}`

- Description：pool 名称，必须唯一
- Type：String
- Required：Yes.

`{pg-num}`

- Description

  The total number of placement groups for the pool. See [Placement Groups](https://ceph.readthedocs.io/en/latest/rados/operations/placement-groups) for details on calculating a suitable number. The default value `8` is NOT suitable for most systems.

- Type：Integer

- Required：Yes.

- Default：8

`{pgp-num}`

- Description

  The total number of placement groups for placement purposes. This **should be equal to the total number of placement groups**, except for placement group splitting scenarios.

- Type：Integer

- Required：Yes. Picks up default or Ceph configuration value if not specified.

- Default：8

`{replicated|erasure}`

- Description

  The pool type which may either be **replicated** to recover from lost OSDs by keeping multiple copies of the objects or **erasure** to get a kind of [generalized RAID5](https://ceph.readthedocs.io/en/latest/rados/operations/erasure-code) capability. The **replicated** pools require more raw storage but implement all Ceph operations. The **erasure** pools require less raw storage but only implement a subset of the available operations.

- Type：String

- Required：No.

- Default：replicated

`[crush-rule-name]`

- Description

  The name of a CRUSH rule to use for this pool. The specified rule must exist.

- Type：String

- Required：No.

- Default

  For **replicated** pools it is the rule specified by the `osd pool default crush rule` config variable. This rule must exist. For **erasure** pools it is `erasure-code` if the `default` [erasure code profile](https://ceph.readthedocs.io/en/latest/rados/operations/erasure-code-profile) is used or `{pool-name}` otherwise. This rule will be created implicitly if it doesn’t exist already.

`[erasure-code-profile=profile]`

- Description

  For **erasure** pools only. Use the [erasure code profile](https://ceph.readthedocs.io/en/latest/rados/operations/erasure-code-profile). It must be an existing profile as defined by **osd erasure-code-profile set**.

- Type：String

- Required：No.

`--autoscale-mode=<on,off,warn>`

- Description

  Autoscale mode

- Type：String

- Required：No.

- Default

  The default behavior is controlled by the `osd pool default pg autoscale mode` option.

If you set the autoscale mode to `on` or `warn`, you can let the system autotune or recommend changes to the number of placement groups in your pool based on actual usage. If you leave it off, then you should refer to [Placement Groups](https://ceph.readthedocs.io/en/latest/rados/operations/placement-groups) for more information.

`[expected-num-objects]`

- Description

  The expected number of objects for this pool. By setting this value ( together with a negative **filestore merge threshold**), the PG folder splitting would happen at the pool creation time, to avoid the latency impact to do a runtime folder splitting.

- Type：Integer

- Required：No.

- Default：0, no splitting at the pool creation time.





## 将 Pool 关联到应用程序

池在使用前需要与应用程序关联。将与CephFS一起使用的池或RGW自动创建的池自动关联。打算与RBD一起使用的池应该使用rbd工具进行初始化（有关更多信息，请参见[Block Device Commands](https://ceph.readthedocs.io/en/latest/rbd/rados-rbd-cmds/#create-a-block-device-pool) ）。

对于其他情况，您可以手动将自由格式的应用程序名称与池关联：

```bash
$ ceph osd pool application enable {pool-name} {application-name}
```

> CephFS使用应用程序名称cephfs，RBD使用应用程序名称rbd，而RGW使用应用程序名称rgw。



## 设置配额

可以将池配额设置为每个池的最大字节数和/或最大对象数：

```
ceph osd pool set-quota {pool-name} [max_objects {obj-count}] [max_bytes {bytes}]
```

例如：

```bash
$ ceph osd pool set-quota data max_objects 10000
```

要删除配额，请将其值设置为0。



## 删除 Pool

```bash
$ ceph osd pool delete {pool-name} [{pool-name} --yes-i-really-really-mean-it]
```

要删除池，必须在Monitor的配置中将 `mon_allow_pool_delete` 标志设置为 true 。否则，他们将拒绝删除池。

如果您为自己创建的池创建了自己的规则，则在不再需要池时应考虑删除它们：

```bash
$ ceph osd pool get {pool-name} crush_rule
```

例如，如果规则是“ 123”，则可以像这样检查其他池：

```bash
$ ceph osd dump | grep "^pool" | grep "crush_rule 123"
```

如果没有其他池使用该自定义规则，则可以从群集中删除该规则。

如果创建的用户严格地具有不再存在的池的权限，则也应考虑删除这些用户：

```bash
$ ceph auth ls | grep -C 5 {pool-name}
$ ceph auth del {user}
```



## 重命名 Pool

```bash
$ ceph osd pool rename {current-pool-name} {new-pool-name}
```

如果您重命名池，并且具有已通过身份验证的用户的每个池功能，则必须使用新的池名称更新该用户的功能（即 caps）



## 获取 Pool 的统计信息

```bash
$ rados df
```

此外，要获取特定池或全部池的 I/O 信息，请执行以下操作：

```bash
$ ceph osd pool stats [{pool-name}]
```



## 制作 Pool 快照

```bash
$ ceph osd pool mksnap {pool-name} {snap-name}
```



## 删除快照

```bash
$ ceph osd pool rmsnap {pool-name} {snap-name}
```



## 配置 Pool

```bash
$ ceph osd pool set {pool-name} {key} {value}
```

关于 key 和 value 请看：https://ceph.readthedocs.io/en/latest/rados/operations/pools/#set-pool-values

## 获取 Pool 配置

```bash
$ ceph osd pool get {pool-name} {key}
```

关于 key 和 value 请看：https://ceph.readthedocs.io/en/latest/rados/operations/pools/#get-pool-values

## 设置复制数

```bash
$ ceph osd pool set {poolname} size {num-replicas}
```

>{num-replicas}包含对象本身。如果要使用该对象和该对象的两个副本（总共三个对象），请指定3。

例如：

```bash
$ ceph osd pool set data size 3
```

您可以为每个池执行此命令。注意：对象可能在降级模式下接受的 I/O 数量少于池大小的副本。要为 I/O 设置最小数量的必需副本，应使用min_size设置。例如：

```bash
$ ceph osd pool set data min_size 2
```

这样可以确保数据池中的任何对象都不会收到少于min_size个副本的I / O。





## 获取对象的副本数

```bash
$ ceph osd dump | grep 'replicated size'
```









