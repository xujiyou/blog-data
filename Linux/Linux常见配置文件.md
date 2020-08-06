---
title: Linux常见配置文件
date: 2020-07-13 15:32:06
tags:
---

Linux 的配置文件按道理应全部放到 /etc 目录。但是这里也记录一些 Linux 中的关键文件

## DNS 配置文件

`/etc/resolv.conf` 文件保存有 DNS 的配置。



## 初始化配置文件

`/etc/rc.local` 这个文件用于开启启动一些服务，也可以添加一些 mount 挂载命令，在开启启动时进行挂载。



## 网络命名空间目录

`/var/run/netns/` 中



## 调整文件描述符数量

显示当前文件描述符数量：

```
ulimit -n
```

在 `/etc/security/limits.conf` 最后添加如下两行：

```
* soft nofile 605536
* hard nofile 605536
```

重启当前命令行生效