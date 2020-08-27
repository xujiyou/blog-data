# Ceph 网络配置

官方文档：https://ceph.readthedocs.io/en/latest/rados/configuration/network-config-ref/

网络配置对于构建高性能Ceph存储群集至关重要。Ceph存储群集不代表Ceph客户端执行请求路由或调度。相反，Ceph客户端直接向Ceph OSD守护程序发出请求。Ceph OSD守护程序代表Ceph客户端执行数据复制，这意味着复制和其他因素会在Ceph存储群集网络上施加额外的负载。

我们的快速入门配置提供了一个简单的Ceph配置文件，该文件仅设置监视器IP地址和守护程序主机名。除非您指定群集网络，否则Ceph将假定为单个“公共”网络。Ceph只能在公共网络上正常运行，但是大型集群中的第二个“集群”网络可能会显着改善性能。

可以使用两个网络来运行Ceph存储群集：一个公共（前端）网络和一个群集（后端）网络。但是，这种方法使网络配置（包括硬件和软件）复杂化，并且通常不会对整体性能产生重大影响。因此，我们通常建议双NIC系统在同一网络上配置两个IP或绑定。

如果尽管很复杂，但仍然希望使用两个网络，则每个Ceph节点将需要具有多个NIC。有关其他详细信息，请参见硬件建议-网络。

> 注：NIC (Network Interface Card）即网卡。



## IPTABLES

默认情况下，守护程序绑定到6800：7300范围内的端口。您可以自行决定配置此范围。在配置IP表之前，请检查默认iptables配置：

```bash
$ sudo iptables -L
```

某些Linux发行版包含一些规则，这些规则拒绝所有入站请求（除来自所有网络接口的SSH外）。例如：

```
REJECT all -- anywhere anywhere reject-with icmp-host-prohibited
```

首先，您将需要在公共网络和群集网络上都删除这些规则，并在准备加强Ceph节点上的端口时将它们替换为适当的规则。



#### MONITOR IP TABLES

默认情况下，Ceph Monitors监听端口3300和6789。另外，Ceph Monitors始终在公共网络上运行。使用以下示例添加规则时，请确保将{iface}替换为公共网络接口（例如eth0，eth1等），将{ip-address}替换为公共网络的IP地址，并将{netmask}与用于公共网络的网络掩码：

```bash
$ sudo iptables -A INPUT -i {iface} -p tcp -s {ip-address}/{netmask} --dport 6789 -j ACCEPT
```



#### MDS AND MANAGER IP TABLES

Ceph Metadata Server 或 Ceph Manager 侦听公用网络上从端口 6800 开始的第一个可用端口。请注意，此行为不是确定性的，因此，如果您在同一主机上运行多个OSD或MDS，或者如果在短时间内重启守护程序，则这些守护程序将绑定到更高的端口。默认情况下，您应该打开整个 6800-7300 系列的端口：

```bash
$ sudo iptables -A INPUT -i {iface} -m multiport -p tcp -s {ip-address}/{netmask} --dports 6800:7300 -j ACCEPT
```



#### OSD IP TABLES

默认情况下，Ceph OSD 和 MGR 、MDS 是一样的，也是占用 6800-7300 之间的端口。

一个Ceph节点上的每个Ceph OSD守护程序最多可以使用四个端口：

- 一个端口用于与客户端和 Monitors 对话。
- 一个端口用于将数据发送到其他 OSD 。
- 两个端口用于每个接口上的心跳。

```bash
$ sudo iptables -A INPUT -i {iface}  -m multiport -p tcp -s {ip-address}/{netmask} --dports 6800:7300 -j ACCEPT
```

> 如果在与Ceph OSD守护程序相同的Ceph节点上运行Ceph元数据服务器，则可以合并公共网络配置步骤。



## Ceph 网络

要配置 Ceph 网络，必须将网络配置添加到配置文件的 [global] 部分。我们5分钟的快速入门提供了一个简单的 Ceph 配置文件，该文件假定一个公共网络，客户端和服务器位于同一网络和子网上。Ceph 只能在公共网络上正常运行。但是，Ceph 允许您建立更具体的标准，包括公共网络的多个 IP 网络和子网掩码。您还可以建立一个单独的群集网络来处理 OSD 心跳，对象复制和恢复流量。不要将您在配置中设置的IP地址与网络客户端可以用来访问您的服务的面向公众的IP地址混淆。典型的内部 IP 网络通常为 192.168.0.0 或 10.0.0.0 。

> 如果为公用网络或群集网络指定了多个IP地址和子网掩码，则网络内的子网必须能够相互路由。此外，请确保在IP表中包括每个IP地址/子网，并在必要时为其打开端口。
>
> Ceph对子网使用CIDR表示法（例如10.0.0.0/24）。

配置网络后，可以重新启动集群或重新启动每个守护程序。 Ceph守护程序是动态绑定的，因此，如果您更改网络配置，则不必立即重新启动整个集群。



#### 公共网络

要配置公共网络，请将以下选项添加到Ceph配置文件的[global]部分：

```ini
[global]
        # ... elided configuration
        public network = {public-network/netmask}
```



#### 集群网络

如果声明群集网络，则OSD将在群集网络上路由心跳，对象复制和恢复流量。与使用单个网络相比，这可以提高性能：

```ini
[global]
        # ... elided configuration
        cluster network = {cluster-network/netmask}
```

建议从公共网络或Internet无法访问群集网络以增加安全性。





## Ceph 守护程序

每个监视守护程序都配置为绑定到特定的IP地址：

```ini
[global]
    mon host = 10.0.0.2, 10.0.0.3, 10.0.0.4
```

mon主机值可以是IP地址列表，也可以是通过DNS查找的名称。对于具有多个A或AAAA记录的DNS名称，将对所有记录进行探测以发现监视器。一旦达到一台 Monitor，便会发现所有其他当前 Monitor ，因此mon主机配置选项仅需要足够最新，以便客户端可以访问当前在线的一台Monitor。

MGR，OSD和MDS守护程序将绑定到任何可用地址，并且不需要任何特殊配置。

但是，可以为它们指定一个特定的IP地址，以便与公共地址（和/或OSD守护程序，群集地址）配置选项绑定。例如：

```ini
[osd.0]
        public addr = {host-public-ip-address}
        cluster addr = {host-cluster-ip-address}
```

> 对于两个网络集群中，只有一个 OSD 网卡：
>
> 通常，我们不建议在具有两个网络的群集中部署具有单个NIC的OSD主机。但是，您可以通过在Ceph配置文件的[osd.n]部分添加一个公共addr条目来强迫OSD主机在公共网络上运行来完成此操作，其中n表示具有一个NIC的OSD的编号。此外，公共网络和群集网络必须能够相互路由流量，出于安全原因，我们不建议这样做。



## 网络配置设置

不需要网络配置设置。除非您专门配置群集网络，否则Ceph假定所有主机都在公共网络上运行。



#### 公共网络

公共网络配置允许您专门定义公共网络的IP地址和子网。您可以专门为特定守护程序分配静态 IP 地址或使用 public addr 设置覆盖公共网络设置。

`public network`

- Description

  公用（前端）网络的IP地址和子网掩码（例如192.168.0.0/24）。在[global]中设置。您可以指定逗号分隔的子网。

- Type：`{ip-address}/{netmask} [, {ip-address}/{netmask}]`

- Required：No

- Default：N/A

`public addr`

- Description

  公共（前端）网络的IP地址。为每个守护程序设置。

- Type：IP Address

- Required：No

- Default：N/A



## 集群网络

群集网络配置允许您声明群集网络，并专门为群集网络定义IP地址和子网。您可以使用特定 OSD 守护程序的群集地址设置来专门分配静态 IP地址或覆盖群集网络设置。

`cluster network`

- Description

  群集（后端）网络的IP地址和子网掩码（例如10.0.0.0/24）。在[global]中设置。您可以指定逗号分隔的子网。

- Type：`{ip-address}/{netmask} [, {ip-address}/{netmask}]`

- Required：No

- Default：N/A

`cluster addr`

- Description

  群集（后端）网络的IP地址。为每个守护程序设置。

- Type：Address

- Required：No

- Default：N/A



## Bind

绑定设置可设置Ceph OSD和MDS守护进程使用的默认端口范围。默认范围是6800：7300。确保您的IP表配置允许您使用配置的端口范围。

您还可以使Ceph守护程序绑定到IPv6地址而不是IPv4地址。

`ms bind port min`

- Description

  OSD或MDS守护程序将绑定到的最小端口号。

- Type：32-bit Integer

- Default：`6800`

- Required：No

`ms bind port max`

- Description

  OSD或MDS守护程序将绑定到的最大端口号。

- Type：32-bit Integer

- Default：`7300`

- Required：No.

`ms bind ipv6`

- Description

  使Ceph守护程序能够绑定到IPv6地址。当前，该Messenger使用IPv4或IPv6，但不能同时使用两者。

- Type：Boolean

- Default：`false`

- Required：No

`public bind addr`

- Description

  在某些动态部署中，Ceph MON守护程序可能会本地绑定到一个IP地址，该IP地址与广告给网络中其他对等方的公共地址不同。环境必须确保正确设置了路由规则。如果设置了public bind addr，则Ceph MON守护进程将在本地绑定到它，并在monmap中使用public addr将其地址通告给对等方。此行为仅限于MON守护程序。

- Type：IP Address

- Required：No

- Default：N/A



## TCP

默认情况下，Ceph禁用TCP缓冲。

`ms tcp nodelay`

- Description

  Ceph启用 `ms tcp nodelay`，以便立即发送每个请求（不缓冲）。停用Nagle的算法会增加网络流量，这可能会导致延迟。如果遇到大量小数据包，则可以尝试禁用ms tcp nodelay。

- Type：Boolean

- Required：No

- Default：`true`

`ms tcp rcvbuf`

- Description

  网络连接的接收端上的套接字缓冲区的大小。默认禁用。

- Type：32-bit Integer

- Required：No

- Default`0`

`ms tcp read timeout`

- Description

  如果客户端或守护进程向另一个Ceph守护进程发出请求，并且没有丢弃未使用的连接，则ms tcp读取超时会将连接定义为在指定的秒数后为空闲状态。

- Type：Unsigned 64-bit Integer

- Required：No

- Default：`900` 15 minutes.











