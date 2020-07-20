---
title: Linux性能分析
date: 2020-07-13 13:59:50
tags:
---

Linux 系统层面的性能分析，一般包括这几个方面：CPU、内存、磁盘IO、网络、文件系统等。另外还包括一些内核参数调优、优先级等。

需要掌握一些工具的使用，帮助我们分析。

参考下面几张图



##  Linux observability tools | Linux 性能观测工具

![img](../../resource/v2-a3aa9d41686557c0c7ec9304500277e2_1440w.png)

首先学习的Basic Tool有如下： uptime、top(htop)、mpstat、isstat、vmstat、free、ping、nicstat、dstat.

高级的命令如下： sar、netstat、pidstat、strace、tcpdump、blktrace、iotop、slabtop、sysctl、/proc。

还有一张图：

![img](../../resource/v2-d65908c900ed43429a56dd90d5ba201e_1440w.png)



## Linux benchmarking tools | Linux 性能测评工具

![img](../../resource/v2-84bc64e3f374f930edf88d24df4311f6_1440w.png)



## Linux tuning tools | Linux 性能调优工具

![img](../../resource/v2-33ac42072fa872d232a79a6b7a723988_1440w.png)

## Linux observability sar | linux性能观测工具

![img](../../resource/v2-52d350253a637fd1eb44bfc4d7e7bca7_1440w.png)

sar（System Activity Reporter系统活动情况报告）是目前LINUX上最为全面的系统性能分析工具之一，可以从多方面对系统的活动进行报告，包括：文件的读写情况、系统调用的使用情况、磁盘I/O、CPU效率、内存使用状况、进程活动及IPC有关的活动等方面。





