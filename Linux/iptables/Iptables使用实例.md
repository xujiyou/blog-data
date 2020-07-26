---
title: Iptables使用实例
date: 2020-07-20 10:31:10
tags:
---

下面实战学一下 Iptables。

## 安装

```bash
$ yum install iptables-services
```

启动：

```bash
$ systemctl enable iptables
$ systemctl start iptables
```

Iptables 的配置文件是 `/etc/sysconfig/iptables` ，它的初始内容如下：

```bash
*filter # filter 表
:INPUT ACCEPT [0:0] # 该规则表示INPUT表默认策略是ACCEP
:FORWARD ACCEPT [0:0] # 该规则表示FORWARD表默认策略是ACCEPT
:OUTPUT ACCEPT [0:0] # 该规则表示OUTPUT表默认策略是ACCEPT
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT # 意思是允许进入的数据包只能是刚刚我发出去的数据包的回应，ESTABLISHED：已建立的链接状态。RELATED：该数据包与本机发出的数据包有关。
-A INPUT -p icmp -j ACCEPT # 接受 icmp 请求
-A INPUT -i lo -j ACCEPT # 意思就允许本地环回接口在INPUT表的所有数据通信，-i 参数是指定接口，接口是lo，lo就是Loopback（本地环回接口）
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT # 接受 22 端口的 TCP 连接 

# 下面这两条的意思是在INPUT表和FORWARD表中拒绝所有其他不符合上述任何一条规则的数据包。并且发送一条host prohibited 的消息给被拒绝的主机。其他开放相关的规则应该放在这两条规则上边。
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
COMMIT
```



## 禁止 ICMP

在禁止之前进行ping：

```bash
$ ping 192.168.98.131
```

可以ping通

在 `/etc/sysconfig/iptables` 去掉下面这句：

```bash
-A INPUT -p icmp -j ACCEPT
```

并进行重启（注意不要进行 service iptables save，因为这里是修改的配置文件）：

````bash
$ service iptables restart
````

再次 ping ：

```bash
$ ping 192.168.98.131
```

发现 ping 不通了。



## 开放端口

想开放 3306 端口，供外部访问 mysql。

在 `/etc/sysconfig/iptables` 文件中的 filter 表中加入：

```
-A INPUT -p tcp -m state --state NEW -m tcp --dport 3306 -j ACCEPT
```

重启 iptables：

```bash
$ service iptables restart
```

在外部测试：

```bash
$ telnet 192.168.98.131 3306
```





## Nat 端口转发实战

有三台机器：

```
192.168.112.152
192.168.112.153
192.168.112.154
```

192.168.112.152 上的 3306 端口有 mysql 服务。

实现将（192.168.112.153:7410）端口流量转发给（192.168.112.152:3306）。

```bash
$ iptables -t nat -A PREROUTING -d 192.168.112.153/32 -p tcp --dport 7410 -j DNAT --to-destination 192.168.112.152:3306;
$ iptables -t nat -A POSTROUTING -d 192.168.112.152/32 -p tcp --dport 3306 -j SNAT --to-source 192.168.112.153;
```

DNAT 的意思是修改目的地址，将 本机接收到的目的地址是 192.168.112.153:7410 的连接修改成目的地址是 192.168.112.152:3306 的连接

SNAT 的意思是修改源地址，这里将去往 192.168.112.152:3306 的源地址改为本机的 192.168.112.153。

netfilter 框架会对设置的每条一条（出向或入向）规则，自动设置它的反向规则，因此我们只需要设 置一个方向的规则即可。

保存：

```bash
$ service iptables save
$ service iptables restart
```

测试，在 192.168.112.154 上执行：

```bash
$ telnet 192.168.112.153 7410
$ telnet 192.168.112.152 3306
```

这两条命令的输出是一样的，说明达到了目的。