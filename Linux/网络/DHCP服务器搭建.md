---
title: DHCP服务器搭建
date: 2020-07-13 16:30:59
tags:
---

准备在 Linux 主机上搭建一个 HDCP 服务器，给其上的 KVM 虚拟机使用。



## DHCP 服务器安装

安装：

```bash
$ yum install dhcp -y
```

dhcp 的配置文件为 `/etc/dhcp/dhcpd.conf` 

在 `/usr/share/doc/dhcp-4.2.5/dhcpd.conf.example` 中有一份示例。可以拷贝过去：

```bash
$ cp /usr/share/doc/dhcp-4.2.5/dhcpd.conf.example  /etc/dhcp/dhcpd.conf 
```



## 修改配置文件

接  [使用linux网桥及vlan实现交换机.md](使用linux网桥及vlan实现交换机.md) 这篇文章，我想实现为 192.168.30.1/24 网段的虚拟机自动分配IP地址。

往配置文件的 `log-facility local7;` 下面加入配置：

```
subnet 192.168.30.0 netmask 255.255.255.0 {
  range 192.168.30.10 192.168.30.253;
}
```



## 启动

启动 DHCP 服务：

```bash
$ systemctl start dhcpd.service  
$ systemctl enable dhcpd.service  
```



## 测试

在宿主机上启动 DHCP 服务器后，虚拟机上的网卡（即DHCP客户端）会进行广播 DHCP 请求，宿主机得到请求后会响应虚拟机的请求，从而完成 IP 地址的设置。

在虚拟机上先删除 IP 地址：

```bash
$ ip addr del 192.168.30.3/24 dev eth0
```

网卡的配置文件 `/etc/sysconfig/network-scripts/ifcfg-eth0` ：

```
ONBOOT=yes
BOOTPROTO=dhcp
```

然后重启网络：

```bash
$ systemctl restart network
```

重启完成后，虚拟机就有从 DHCP 服务器获取到的 IP 地址了。









