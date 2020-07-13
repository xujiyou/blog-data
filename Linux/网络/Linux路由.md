---
title: Linux路由
date: 2020-07-13 15:22:55
tags:
---

Linux 路由需要记住两点：跨网段通信需要经过路由；Linux 本身就是一台路由器。



## 开启路由功能

```
$ cat /proc/sys/net/ipv4/ip_forward
```

如果值为 1，表示开启了路由功能，如未开启，需要在 `/etc/sysctl.conf` 中设置：

```
net.ipv4.ip_forward = 1
```

然后执行 `sysctl -p` 使之生效。



## 实践

