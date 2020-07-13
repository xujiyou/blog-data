---
title: Linux常见配置文件
date: 2020-07-13 15:32:06
tags:
---

Linux 的配置文件按道理应全部放到 /etc 目录。

## DNS 配置文件

`/etc/resolv.conf` 文件保存有 DNS 的配置。



## 初始化配置文件

`/etc/rc.local` 这个文件用于开启启动一些服务，也可以添加一些 mount 挂载命令，在开启启动时进行挂载。