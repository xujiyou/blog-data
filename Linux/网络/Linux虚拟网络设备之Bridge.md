---
title: Linux虚拟网络设备之Bridge
date: 2020-07-03 13:40:00
tags:
---



## 什么是bridge？
首先，bridge是一个虚拟网络设备，所以具有网络设备的特征，可以配置IP、MAC地址等；其次，bridge是一个虚拟交换机，和物理交换机有类似的功能。

对于普通的网络设备来说，只有两端，从一端进来的数据会从另一端出去，如物理网卡从外面网络中收到的数据会转发给内核协议栈，而从协议栈过来的数据会转发到外面的物理网络中。

而bridge不同，bridge有多个端口，数据可以从任何端口进来，进来之后从哪个口出去和物理交换机的原理差不多，要看mac地址。



## 常用操作

安装工具：

```bash
$ yum install bridge-utils
```

查看网桥列表：

```bash
$ brctl show
```

docker0 就是一个虚拟网桥

查看网络命名空间：

```
$ ip netns
$ ls /var/run/netns
```

在某个命名空间中查看网卡：

```bash
$ ip netns exec qdhcp-a5c62f47-b1c1-4a8e-a20d-81371017843a ip a
```

查看网关：

```
$ netstat -rn
```



## 创建 Bridge

使用 iproute2 命令创建 Bridge：

```bash
$ ip link add name br0 type bridge
$ ip link set br0 up
```

当刚创建一个bridge时，它是一个独立的网络设备，只有一个端口连着协议栈，其它的端口啥都没连，这样的bridge没有任何实际功能。

```
+----------------------------------------------------------------+
|                                                                |
|       +------------------------------------------------+       |
|       |             Newwork Protocol Stack             |       |
|       +------------------------------------------------+       |
|              ↑                                ↑                |
|..............|................................|................|
|              ↓                                ↓                |
|        +----------+                     +------------+         |
|        |   eth0   |                     |     br0    |         |
|        +----------+                     +------------+         |
| 172.20.21.16 ↑                                                 |
|              |                                                 |
|              |                                                 |
+--------------|-------------------------------------------------+
               ↓
         Physical Network
```



## 将bridge和veth设备相连

创建一对veth设备，并配置上IP:

```bash
$ ip link add veth0 type veth peer name veth1
$ ip netns add xujiyou
$ ip link set veth1 netns xujiyou
$ ip addr add 192.168.3.101/24 dev veth0
$ ip netns exec xujiyou ip addr add 192.168.3.102/24 dev veth1
$ ip link set veth0 up
$ ip netns exec xujiyou ip link set veth1 up
```

这时候 192.168.3.101 和 192.168.3.102 都是可以 ping 的通的。

```
$ ping -c 1 -I veth0 192.168.3.102
$ ip netns exec xujiyou ping -c 2 -I veth1 192.168.3.101
```

Veth设备对必须在不同的命名空间才能互相ping通（CentOS 7）

将 veth0 连上 bridge：

```bash
$ ip link set dev veth0 master br0
```

查看 bridge 连接：

```bash
$ bridge link
```

这时候网络就是这个样子：

```
+----------------------------------------------------------------+
|                                                                |
|       +------------------------------------------------+       |
|       |             Newwork Protocol Stack             |       |
|       +------------------------------------------------+       |
|            ↑            ↑              |            ↑          |
|............|............|..............|............|..........|
|            ↓            ↓              ↓            ↓          |
|  +------------+     +--------+     +-------+    +-------+      |
|  |172.20.21.16|     |        |     | .3.101|    | .3.102|      |
|  +------------+     +--------+     +-------+    +-------+      |
|  |   eth0     |     |   br0  |<--->| veth0 |    | veth1 |      |
|  +------------+     +--------+     +-------+    +-------+      |
|            ↑                           ↑            ↑          |
|            |                           |            |          |
|            |                           +------------+          |
|            |                                                   |
+------------|---------------------------------------------------+
             ↓
     Physical Network
```



br0和veth0相连之后，发生了几个变化：

- br0和veth0之间连接起来了，并且是双向的通道
- 协议栈和veth0之间变成了单通道，协议栈能发数据给veth0，但veth0从外面收到的数据不会转发给协议栈
- br0的mac地址变成了veth0的mac地址

相当于bridge在veth0和协议栈之间插了一脚，在veth0上面做了点小动作，将veth0本来要转发给协议栈的数据给拦截了，全部转发给bridge了，同时bridge也可以向veth0发数据。

下面来检验一下是不是这样的：

```bash
$ ping -c 1 -I veth0 192.168.3.102
```

这时发现 ping 不通了，为什么ping 不通了那，可以抓包看下：

```
$ ip netns exec xujiyou tcpdump -n -i veth1
14:55:28.757217 ARP, Request who-has 192.168.3.102 tell 192.168.3.101, length 28
14:55:28.757231 ARP, Reply 192.168.3.102 is-at 6e:17:0c:c1:10:1b, length 28

$ tcpdump -n -i veth0
14:55:28.757203 ARP, Request who-has 192.168.3.102 tell 192.168.3.101, length 28
14:55:28.757232 ARP, Reply 192.168.3.102 is-at 6e:17:0c:c1:10:1b, length 28

$ tcpdump -n -i br0
14:55:28.757232 ARP, Reply 192.168.3.102 is-at 6e:17:0c:c1:10:1b, length 28
```

从上面的抓包可以看出，去和回来的流程都没有问题，问题就出在veth0收到应答包后没有给协议栈，而是给了br0，于是协议栈得不到veth1的mac地址，从而通信失败。

 



## 给bridge配上IP

通过上面的分析可以看出，给veth0配置IP没有意义，因为就算协议栈传数据包给veth0，应答包也回不来。这里我们就将veth0的IP让给bridge。

```
$ ip addr del 192.168.3.101/24 dev veth0
$ ip addr add 192.168.3.101/24 dev br0
```

现在网络就是下面这样了：

```
+----------------------------------------------------------------+
|                                                                |
|       +------------------------------------------------+       |
|       |             Newwork Protocol Stack             |       |
|       +------------------------------------------------+       |
|            ↑            ↑                           ↑          |
|............|............|...........................|..........|
|            ↓            ↓                           ↓          |
|        +------+     +--------+     +-------+    +-------+      |
|        | .3.21|     | .3.101 |     |       |    | .3.102|      |
|        +------+     +--------+     +-------+    +-------+      |
|        | eth0 |     |   br0  |<--->| veth0 |    | veth1 |      |
|        +------+     +--------+     +-------+    +-------+      |
|            ↑                           ↑            ↑          |
|            |                           |            |          |
|            |                           +------------+          |
|            |                                                   |
+------------|---------------------------------------------------+
             ↓
     Physical Network
```

再通过网桥去 ping，发现成功了：

```bash
$ ping -c 1 -I br0 192.168.3.102
```

但ping网关还是失败，因为这个bridge上只有两个网络设备，分别是192.168.3.101和192.168.3.102，br0不知道192.168.3.1在哪。



## 将物理网卡添加到bridge

br0根本不区分接入进来的是物理设备还是虚拟设备，对它来说都一样的，都是网络设备，所以当eth0加入br0之后，落得和上面veth0一样的下场，从外面网络收到的数据包将无条件的转发给br0，自己变成了一根网线。

这时通过eth0来ping网关失败，但由于br0通过eth0这根网线连上了外面的物理交换机，所以连在br0上的设备都能ping通网关，这里连上的设备就是veth1和br0自己，veth1是通过veth0这根网线连上去的，而br0可以理解为自己有一块自带的网卡。

添加网桥：

```bash
$ brctl addbr br0
```

将eth0接口加入此网桥：

```bash
$ brctl addif br0 eth0 
```

去除 eth0 的地址：

```bash
$ ifconfig eth0 0.0.0.0
```

为 br0 添加地址：

```bash
$ ifconfig br0 172.20.21.16 netmask 255.255.252.0
```

增加网关：

```bash
$ route add default gw 172.20.23.254 dev br0
```

网桥的地址就是为了可以方面进行ssh登录宿主机。与网桥连接的虚拟机IP地址可以设置为网桥处于同一个ip地址网段，也可是设置为不相同的ip地址。

现在通过 SSH 就可以连接 Linux 了。

查看网桥：

```
$ brctl show
```



## 持久化设置

上面将物理网卡添加到网桥的设置，在主机重启后，配置就没了，为了持久化，可以写入网络配置文件。

添加一个网桥：

```bash
$ vim /etc/sysconfig/network-scripts/ifcfg-br0
```

内容如下：

```
DEVICE=br0
TYPE=Bridge
ONBOOT=yes
NM_CONTROLLED=yes
BOOTPROTO=static
IPADDR=192.168.98.120
NETMASK=255.255.255.0
GATEWAY=192.168.98.1
DNS1=10.28.100.100
```

配置物理网卡：

```bash
$ vim /etc/sysconfig/network-scripts/ifcfg-enp2s0
```

内容如下：

```
TYPE=Ethernet
DEVICE=enp2s0
ONBOOT=yes
UUID=d4e8208e-a35b-4944-87e7-c4203a00f8eb
BRIDGE=br0
NM_CONTROLLED=yes
```

注意网卡配置文件里面的内容，每一行的结尾不要有其他空格等字符。

重启网络：

```bash
$ systemctl restart network
```

查看网络：

```bash
$ ip addr
$ brctl show
$ ping www.baidu.com
```















