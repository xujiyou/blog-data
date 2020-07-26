---
title: Nat教程
date: 2020-07-26 13:51:58
tags:
---

在 iptables 中，`nat` table 被细分成了 `DNAT` （修改目的地址） 和 `SNAT`（修改源地址）

### SNAT - 修改源 IP 为固定新 IP （静态）

前面的将本地私有网络连接到因特网的例子中： [Iptables使用实例.md](Iptables使用实例.md) ，我们已经使用了 Source NAT（SNAT ）。如名字所暗示，发送方的地址会被静态地修改。

在例子中我们选择 `MASQUERADE` 的原因在于：**对于 SNAT，必须显式指定转换后的 IP**。 如果路由器配置的是静态 IP 地址，那 SNAT 是最合适的选择，因为它比 `MASQUERADE` 更 快，后者对每个包都需要检查指定的输出端口上配置的 IP 地址。

**因为 SNAT 只对离开路由器的包有意义，因此它只用在 `POSTROUTING` chain 中。**



### MASQUERADE - 修改源 IP 为动态新 IP（动态获取网络接口 IP）

和 `SNAT` 类似，但是对每个包都会动态获取指定输出接口（网卡）的 IP，因此如果接口 的 IP 地址发送了变化，`MASQUERADE` 规则不受影响，可以正常工作；而对于 `SNAT` 就必须重新调整规则。

和 `SNAT` 一样，`MASQUERADE` 只对 `POSTROUTING` chain 有意义。但和 `SNAT` 不同， `MASQUERADE` 不支持更详细的配置项了。



### DNAT - 修改目的 IP

如果想修改包的目的 IP 地址，那需要使用 Destination NAT（DNAT）。

DNAT 可以用于运行在防火墙后面的服务器。

显然，接收端修改**必须在做路由决策之前，因此 DNAT 适用于 `PRETOUTING` 和 `OUTPUT` （本地生成的包）chain**。



### REDIRECT - 将包重定向到本机另一个端口

REDIRECT 是 DNAT 的一个特殊场景。包被重定向到路由器的另一个本地端口，可以实现， 例如透明代理的功能。和 DNAT 一样，REDIRECT 适用于 `PRETOUTING` 和 `OUTPUT` chain 。



## 另

iptables 好像并不支持  端口多路复用。















