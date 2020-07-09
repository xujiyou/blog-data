---
title: IP隧道
date: 2020-07-06 21:36:32
tags:
---

 tun 是一个网络层的设备，也被叫做点对点设备，之所以叫这个名字，是因为 tun 常常被用来做隧道通信（tunnel）。

## IP 隧道

Linux 原生支持多种三层隧道，其底层实现原理都是基于 tun 设备。我们可以通过命令 `ip tunnel help` 查看 IP 隧道的相关操作。

![image-20200706213914643](../../resource/image-20200706213914643.png)

可以看到，Linux 原生一共支持 5 种 IP 隧道。

- `ipip`：即 `IPv4 in IPv4`，在 IPv4 报文的基础上再封装一个 IPv4 报文。
- `gre`：即通用路由封装（`Generic Routing Encapsulation`），定义了在任意一种网络层协议上封装其他任意一种网络层协议的机制，IPv4 和 IPv6 都适用。
- `sit`：和 `ipip` 类似，不同的是 `sit` 是用 IPv4 报文封装 IPv6 报文，即 `IPv6 over IPv4`。
- `isatap`：即站内自动隧道寻址协议（`Intra-Site Automatic Tunnel Addressing Protocol`），和 `sit` 类似，也是用于 IPv6 的隧道封装。
- `vti`：即虚拟隧道接口（`Virtual Tunnel Interface`），是 cisco 提出的一种 `IPsec` 隧道技术。



## 实践 IPIP 隧道

加载 ipip 模块：

```bash
$ modprobe ipip
$ lsmod | grep ipip
```

加载 `ipip` 模块后，就可以创建隧道了，方法是先创建一个 tun 设备，然后将该 tun 设备绑定为一个 `ipip` 隧道即可。

实验拓扑如下：

![img](../../resource/431521-20190320132302980-826956388.png)











