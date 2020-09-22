---
title: Linux内核参数调优
date: 2020-07-12 18:07:50
tags:
---

熟悉 Linux 参数，不仅可以对内核调优优帮助，还可以更好地理解一些系统原理。



## 常用操作

查看当前系统的所有内核参数：

```bash
$ sysctl -a
```

统计行数：

```bash
$ sysctl -a | wc -l
2485
```

有几千个参数。

查看某一项参数的值：

```bash
$ sysctl kernel.hostname
```

修改内核参数：

```bash
$ sysctl  kernel.hostname=abc
```



## 配置文件

内核参数的配置文件是 `/etc/sysctl.conf` ，可以往这里面写入一些配置，修改后，使用 `sysctl -p` 使之生效。



## 系统配置目录

在 `/proc/sys` 中，有所有的配置，每一项配置都有一个单独的文件，在上面使用命令修改了参数后，这个目录下的文件也会跟着改变。

比如 `kernel.hostname` 的配置文件是 `/proc/sys/kernel/hostname`。

```bash
$ echo "drift-1.cloud.bbdops.com" > /proc/sys/kernel/hostname 
$ sysctl kernel.hostname
```

不过这种修改文件的方式只是临时的，主机重启后就失效了。

关于这个目录，官方还有个文档：https://www.kernel.org/doc/html/latest/admin-guide/sysctl/index.html

这个文档对每一个参数都进行了说明，不过针对的是最新版本的内核。



## 关闭 Swap 

添加内核参数：

```
vm.swappiness = 0
```







