---
title: Ceph储存引擎BlueStore
date: 2020-06-27 20:30:25
tags:
---

BlueStore 是 Ceph 最新的储存引擎，不同于 FileStore，BlueStore 会绕过 Linux 的文件系统，直接接管裸设备（磁盘），直接进行对象操作，不再进行对象与文件之间的转换，从而使得整个对象的 I/O 路径大大缩短，除此之外，BlueStore 还对 SSD 进行了优化。

