---
title: ipvs使用实例
date: 2020-07-26 14:37:31
tags:
---

## LVS

LVS 由2部分程序组成，包括 ipvs 和 ipvsadm。

1. ipvs(ip virtual server)：一段代码工作在内核空间，叫ipvs，是真正生效实现调度的代码。

2. ipvsadm：另外一段是工作在用户空间，叫ipvsadm，负责为ipvs内核框架编写规则，定义谁是集群服务，而谁是后端真实的服务器(Real Server)



## **LVS相关术语**

1. DS：Director Server。指的是前端负载均衡器节点。
2. RS：Real Server。后端真实的工作服务器。 
3. VIP：向外部直接面向用户请求，作为用户请求的目标的IP地址。 
4. DIP：Director Server IP，主要用于和内部主机通讯的IP地址。 
5. RIP：Real Server IP，后端服务器的IP地址。 
6. CIP：Client IP，访问客户端的IP地址。



## 实践LVS的NAT模式

我这里有三台机器：

```
192.168.112.152
192.168.112.153
192.168.112.154
```

准备利用 ipvs 实现，访问 192.168.112.152:81 会负载均衡到 192.168.112.153:10000、192.168.112.154:10000

下面来动手。

director 即 192.168.112.152，real server 是 192.168.112.153 和 192.168.112.154

在 director 上安装ipvsadm：

```bash
$ yum install -y ipvsadm
```

director 的内核参数设置：

```bash
# director 服务器上开启路由转发功能:
echo 1 > /proc/sys/net/ipv4/ip_forward
# 关闭 icmp 的重定向
echo 0 > /proc/sys/net/ipv4/conf/all/send_redirects
echo 0 > /proc/sys/net/ipv4/conf/default/send_redirects
echo 0 > /proc/sys/net/ipv4/conf/ens192/send_redirects
# director设置 nat 防火墙
iptables -t nat -F
iptables -t nat -X
iptables -t nat -A POSTROUTING -s 192.168.112.0/24 -j MASQUERADE
```



在 director 上添加一个服务地址 192.168.112.152:81 的virtual service，协议为tcp，策略为wrr：

```bash
$ ipvsadm -A -t 192.168.112.152:81 -s wrr
```

在 director 上分别添加两个个real server，-m 表示 NAT 模式：

```bash
$ ipvsadm -a -t 192.168.112.152:81  -r 192.168.112.153:10000 -m -w 1
$ ipvsadm -a -t 192.168.112.152:81  -r 192.168.112.154:10000 -m -w 1
```

在 director 上查看创建的server table：

```bash
$ ipvsadm-save -n
-A -t 192.168.112.152:81 -s wrr
-a -t 192.168.112.152:81 -r 192.168.112.153:10000 -m -w 1
-a -t 192.168.112.152:81 -r 192.168.112.154:10000 -m -w 1
```

分别在两个 real server 上启动服务：

```bash
$ cd /tmp
$ echo 153 >index.html  
$ python -m SimpleHTTPServer 10000 

$ cd /tmp
$ echo 154 >index.html  
$ python -m SimpleHTTPServer 10000
```

测试：

```bash
$ curl http://192.168.112.152:81
```

这个命令，会依次返回 153 和 154，说明负载均衡生效了！



## LVS结合keepalive

LVS可以实现负载均衡，但是不能够进行健康检查，比如一个 real server 出现故障，LVS 仍然会把请求转发给故障的 real server 服务器，这样就会导致请求的无效性。keepalive 软件可以进行健康检查，而且能同时实现 LVS 的高可用性，解决 LVS 单点故障的问题，其实 keepalive 就是为 LVS 而生的。



## 删除

```bash
$ ipvsadm -l
$ ipvsadm -D -t xujiyou:distinct
```

或者使用：

```bash
$ ipvsadm -C
```





