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

cluster map 和 placement rule 组合在一起就形成了 crush map。



