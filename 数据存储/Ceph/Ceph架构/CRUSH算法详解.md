---
title: CRUSH算法详解
date: 2020-06-27 18:00:44
tags:
---

参考 《Ceph 设计原理与实现》，非常不错的一本书，一直不知道怎么深入学习 Ceph 原理，现在终于找到门路了。

今天先来学学第一章 CRUSH

## 介绍

CRUSH，全称是 Controlled Replication Under Scalable Hashing，即可扩展的基于哈希的受控副本分布策略。

分布式储存系统考虑以下问题：

- 集群中磁盘会增加也会删除。
- 磁盘的容量有的大有的小。
- 数据要有副本分布在不同的磁盘或机器或机架上。

CRUSH 在其中的作用就是为一个数据找到一个合适的磁盘，并且在磁盘增加和删除时也能动态平衡数据。

根据磁盘的容量的不同，在 Ceph 中，每个磁盘都有一个权重，容量越大，权重越高。CRUSH 根据权重将数据映射到所有储存设备之间。这个过程依赖两个东西：cluster map 和 placement rule。



## 动手

cluster map 和 placement rule 组合在一起就形成了 crush map。

placement rule 可以自定义，Ceph 默认包含一些规则，可以使用以下命令查看：

```bash
$ ceph osd crush tunables --help
```

如果使用 default 规则：

```bash
$ ceph osd crush tunables default
```



获取 crush map：

```bash
$ ceph osd getcrushmap -o crushmap
```

获取到的是二进制格式的，需要使用工具转化成文本：

```bash
$ crushtool -d crushmap -o crushmap.txt
```

我这里获取的内容如下：

```
# begin crush map
tunable choose_local_tries 0
tunable choose_local_fallback_tries 0
tunable choose_total_tries 50
tunable chooseleaf_descend_once 1
tunable chooseleaf_vary_r 1
tunable chooseleaf_stable 1
tunable straw_calc_version 1
tunable allowed_bucket_algs 54

# devices
device 0 osd.0 class hdd
device 1 osd.1 class hdd
device 2 osd.2 class hdd

# types
type 0 osd
type 1 host
type 2 chassis
type 3 rack
type 4 row
type 5 pdu
type 6 pod
type 7 room
type 8 datacenter
type 9 zone
type 10 region
type 11 root

# buckets
host ceph-1 {
    id -3       # do not change unnecessarily
    id -4 class hdd     # do not change unnecessarily
    # weight 0.195
    alg straw2
    hash 0  # rjenkins1
    item osd.0 weight 0.195
}
host ceph-2 {
    id -5       # do not change unnecessarily
    id -6 class hdd     # do not change unnecessarily
    # weight 0.195
    alg straw2
    hash 0  # rjenkins1
    item osd.1 weight 0.195
}
host ceph-3 {
    id -7       # do not change unnecessarily
    id -8 class hdd     # do not change unnecessarily
    # weight 0.195
    alg straw2
    hash 0  # rjenkins1
    item osd.2 weight 0.195
}
root default {
    id -1       # do not change unnecessarily
    id -2 class hdd     # do not change unnecessarily
    # weight 0.586
    alg straw2
    hash 0  # rjenkins1
    item ceph-1 weight 0.195
    item ceph-2 weight 0.195
    item ceph-3 weight 0.195
}

# rules
rule replicated_rule {
    id 0
    type replicated
    min_size 1
    max_size 10
    step take default
    step chooseleaf firstn 0 type host
    step emit
}

# end crush map
```

devices 就是磁盘，types 表示 buckets 有哪些类型，根 bucket 叫 root，其他子 bucket 可以是地区、区域、数据中心、房间、机架、主机等，下面的 buckets 就是当前集群的 bucket 列表，我这里的集群比较简单，就三台主机。这些内容就是 cluster map 的内容了。

最后的 rules 就是 placement rule 的内容。

min_size 和 max_size 规定了副本数量的范围，最小要有一个副本，最多 10 个。

三个 step 表示 CRUSH 算法在为数据选择磁盘时使用到的步骤

- `step take default` 表示从名为 default 的 bucket 开始选。

- `step chooseleaf firstn 0 type host` ，chooseleaf 表示使用容灾域模式，这里还可以是 choose ，表示非容灾；firstn 是两种算法之一，还可以是 indep，区别是会不会返回 none，indep 在找不到时会返回 none，firstn 会将 none 忽略；type 在这里为 host，表示副本必须位于不同主机的磁盘上。

- `step emit` 表示选完了，将结果返回。



以文本的方式修改完 crush map 后，还需编译为二进制的：

```bash
$ crushtool -c crushmap.txt -o crushmap-new
```

在注入集群前，可以 进行测试：

```bash
$ crushtool -i crushmap-new --test --min-x 0 --max-x 9 --num-rep 3 --ruleset 0 --show_mappings
```

x 是输入，可以看作是数据的哈希，num-rep 是副本数量，ruleset 指定 rule id，输出结果是不同的哈希的数据会落在哪些 OSD 上。

结果如下：

```
CRUSH rule 0 x 0 [1,2,0]
CRUSH rule 0 x 1 [2,0,1]
CRUSH rule 0 x 2 [2,1,0]
CRUSH rule 0 x 3 [0,1,2]
CRUSH rule 0 x 4 [1,2,0]
CRUSH rule 0 x 5 [0,1,2]
CRUSH rule 0 x 6 [2,0,1]
CRUSH rule 0 x 7 [1,2,0]
CRUSH rule 0 x 8 [2,0,1]
CRUSH rule 0 x 9 [1,2,0]
```

发现，挺均匀的。

注入集群：

```bash
$ ceph osd setcrushmap -i crushmap-new
```



## 数据重平衡

查看各 bucket 和 devices 的使用量：

```bash
$ ceph osd df tree
```

找到数据较多的 OSD，然后就行调整：

```bash
$ ceph osd reweight 2 0.5
```

这里的 2 是 osd 的 id，0.5 是新权重，这个权重越低，表示 OSD 的磁盘容量越少，命令执行完成后，迁出的数据越多。这里的权重要在 0-1 之间。



## 定制集群布局

定制一个集群布局是建立一个健壮且高可靠的 Ceph 集群的最重要的一个步骤！默认的 Ceph 集群是不知道非交互式组件的，如机架、行和数据中心。在初始化部署后，需要按要求来定制布局。

默认配置是在 root 下只有主机和 OSD。

查看布局：

```bash
$ ceph osd tree
```

添加三个机架：

```bash
$ ceph osd crush add-bucket rack01 rack
$ ceph osd crush add-bucket rack02 rack
$ ceph osd crush add-bucket rack03 rack
```

移动主机到机架下：

```bash
$ ceph osd crush move ceph-1 rack=rack01
$ ceph osd crush move ceph-2 rack=rack02
$ ceph osd crush move ceph-3 rack=rack03
```

移动每一个机架到默认的 root 下：

```bash
$ ceph osd crush move rack01 root=default
$ ceph osd crush move rack02 root=default
$ ceph osd crush move rack03 root=default
```

重新查看布局：

```bash
$ ceph osd tree
```



## 计算 PG 数量

正确计算 PG 数量也是构建一个企业 Ceph 集群中至关重要的一步，因为 PG 在一定程度上提高或者影响储存性能。

计算 PG 数量的公式如下：

> PG总数 = （OSD 总数 * 100） / 最大副本数

结果比如五入到 2 的 N 次幂的值。如果集群有 160 个 OSD，且副本数为 3，这样根据公式得到的 PG 总数是 5333.3，五入后这个值最接近的 2 的 N 次幂的结果就是 8192 个 PG。

每个 Pool 中的 PG 总数应该是

> Pool 中的 PG 总数 = PG 总数 / Pool 数量

PG 的数量只与 OSD 的数量有关，与 OSD 的容量无关。

https://ceph.com/pgcalc/

## 修改 PG 和 PGP

PGP 的值应该与 PG 的总数一致。

获取 PG 和 PGP 的值：

```bash
$ ceph osd pool get test pg_num
$ ceph osd pool get test pgp_num
```

检查副本数量：

```bash
$ ceph osd dump | grep size
```





## 总结

CRUSH 具有计算寻址、高并发、动态数据平衡的功能、可定制的副本策略等特性，能够非常方便的实现去中心化，有效抵御物理结构变化并保证性能随规模呈线性扩展。

因此，Ceph 适合对可扩展性、性能和可靠性都有严格要求的大型分布式系统。











