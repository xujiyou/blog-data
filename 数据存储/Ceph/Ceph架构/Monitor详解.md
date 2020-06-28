---
title: Monitor详解
date: 2020-06-28 20:36:54
tags:
---

Monitor 负责监控整个集群的健康状况，它维护了集群的所有的信息，它储存的信息叫做集群 map，包含 Monitor、OSD、PG、CRUSH 和 MDS map。

客户端在与 Ceph 集群进行数据的增删改查时，先与 Monitor 进行通信，客户端拿到 OSD 的信息后，直接与 osd 守护进程进行通信。



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









