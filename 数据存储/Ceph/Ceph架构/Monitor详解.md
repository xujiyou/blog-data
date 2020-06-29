---
title: Monitor详解
date: 2020-06-28 20:36:54
tags:
---

Monitor 负责监控整个集群的健康状况，它维护了集群的所有的信息，它储存的信息叫做集群 map，包含 Monitor、OSD、PG、CRUSH 和 MDS map。

客户端在与 Ceph 集群进行数据的增删改查时，先与 Monitor 进行通信，客户端拿到集群 map 的信息后，然后根据对象和 pool ID将数据转换为对象，然后将对象和 PG 数量一起经过**散列算法**来计算出对象要存放的 PG ，然后再根据 **CRUSH 算法**来确定 PG 所在的主 OSD 的位置，然后客户端直接与 osd 守护进程进行通信，进行数据的操作，主 OSD 操作完数据后，主 OSD 将执行 CRUSH 算法查找和操作辅助 PG 及其 OSD 的位置来实现数据复制，进而实现高可用。

在这个过程中，Monitor 只负责提供集群 map。。。



## Map

#### monitor map

它维护着 monitor 节点间的端到端的信息，其中包括 Ceph 集群 ID、Monitor 主机名、IP地址和端口等信息。获取 monitor map：

```bash
$ ceph mon dump
```

或者：

```bash
$ monmaptool --print monmap
```

#### osd map

储存着与 pool 相关的信息，如 pool 名字、pool ID、类型、副本数、PG，另外还有状态、权重、主机等信息，查看 osd map：

```bash
$ ceph osd dump
```

#### pg map

储存了 pg 的版本、时间戳、储存的对象数、所在的 OSD等信息，获取之：

```bash
$ ceph pg dump
```

#### CURSH map

包含了 cluster map 和 placement rule ，见： [CRUSH算法详解.md](CRUSH算法详解.md) 

```bash
$ ceph osd crush dump
```

或者：

```bash
$ ceph osd getcrushmap -o crushmap
$ crushtool -d crushmap -o crushmap.txt
```

#### MDS map

与 CephFS 相关的元数据。



## Monitor

Monitor 不为客户端储存和提供数据，它只为客户端和集群内的其他节点提供集群的 Map，客户端和其他集群节点定期与 monitor 确认自己持有的是否是集群最新的 map。

Monitor 是轻量级的守护进程，通常不需要大量的系统资源。

查看状态：

```bash
$ ceph mon stat
```









