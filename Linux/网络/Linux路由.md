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

创建两个网络 namespace：

```bash
$ ip netns add ns1
$ ip netns add ns2
```

创建两对 veth-pair，一端分别挂在两个 namespace 中：

```
$ ip link add v1 type veth peer name v1_r
$ ip link add v2 type veth peer name v2_r

$ ip link set v1 netns ns1
$ ip link set v2 netns ns2
```

分别给两对 veth-pair 端点配上 IP 并启用：

```bash
$ ip a a 10.10.10.1/24 dev v1_r
$ ip l s v1_r up
$ ip a a 10.10.20.1/24 dev v2_r
$ ip l s v2_r up

$ ip netns exec ns1 ip a a 10.10.10.2/24 dev v1
$ ip netns exec ns1 ip l s v1 up
$ ip netns exec ns2 ip a a 10.10.20.2/24 dev v2
$ ip netns exec ns2 ip l s v2 up
```



测试：

```bash
$ ip netns exec ns1 ping 10.10.20.2
```

发现不通。



## 添加路由

查看路由：

```bash
$ ip netns exec ns1 route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
10.10.10.0      0.0.0.0         255.255.255.0   U     0      0        0 v1
```

只有一条直连路由，没有去往 `10.10.20.0/24` 网段的路由，怎么通？那就给它配一条：

```bash
$ ip netns exec ns1 route add -net 10.10.20.0 netmask 255.255.255.0 gw 10.10.10.1
$ ip netns exec ns1 route -n
```

同理也给 ns2 配上去往 `10.10.10.0/24` 网段的路由：

```bash
$ ip netns exec ns2 route add -net 10.10.10.0 netmask 255.255.255.0 gw 10.10.20.1
$ ip netns exec ns2 route -n
```

再次测试，发现可以 ping 通了：

```bash
$ ip netns exec ns1 ping 10.10.20.2
```



## 总结

Linux 本身是一台路由器。

上面的实验使用 namespace 效果和使用虚拟机是一样的，关键是知道有这个功能，知道怎么用就差不多了。











