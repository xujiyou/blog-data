# Ipvsadm 常用命令

- ipvsadm 参数

```
-C 清除表中所有的记录
-A --add-service在服务器列表中新添加一条新的虚拟服务器记录
-t 表示为tcp服务
-u 表示为udp服务
-s --scheduler 使用的调度算法， rr | wrr | lc | wlc | lblb | lblcr | dh | sh | sed | nq 默认调度算法是 wlc
-a --add-server 在服务器表中添加一条新的真实主机记录
-t --tcp-service 说明虚拟服务器提供tcp服务
-u --udp-service 说明虚拟服务器提供udp服务
-r --real-server 真实服务器地址
-m --masquerading 指定LVS工作模式为NAT模式
-w --weight 真实服务器的权值
-g --gatewaying 指定LVS工作模式为直接路由器模式（也是LVS默认的模式）
-i --ipip 指定LVS的工作模式为隧道模式
-p 会话保持时间，定义流量呗转到同一个realserver的会话存留时间
```

例子：`ipvsadm -a -t 192.168.112.10:80 -r 192.168.112.13:80 -m -w 1`

- 清除表中所有的记录

```
ipvsadm -C
```

- 查看 `ipvsadm -Ln`

```
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.112.10:80 wrr
  -> 192.168.112.13:80            Route   1      171        2153      
  -> 192.168.112.14:80            Route   1      150        2148      
  -> 192.168.112.15:80            Route   1      154        2150  
```

> Forward 转发方式，当前是路由转发;
> Weight 权重;
> ActiveConn 当前活跃的连接数;
> InActConn 指非活跃连接数，我们将处于 TCP ESTABLISH 状态以外的连接都称为不活跃连接。例如处于 SYN_RECV 状态的连接，处于 TIME_WAIT 状态的连接等。

- 查看速率 `ipvsadm -Ln --rate`

```
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port                 CPS    InPPS   OutPPS    InBPS   OutBPS
  -> RemoteAddress:Port
TCP  192.168.112.10:80                 54      288        0    31825        0
  -> 192.168.112.13:80                 18       95        0    10384        0
  -> 192.168.112.14:80                 18       96        0    10884        0
  -> 192.168.112.15:80                 18       97        0    10556        0
```

> CPS （current connection rate） 每秒连接数
> InPPS （current in packet rate） 每秒的入包个数
> OutPPS （current out packet rate） 每秒的出包个数
> InBPS （current in byte rate） 每秒入流量（字节）
> OutBPS （current out byte rate） 每秒入流量（字节）

- 查看全部连接数和流量 `ipvsadm -Ln --stats`

```
IP Virtual Server version 1.2.1 (size=1048576)
Prot LocalAddress:Port Conns InPkts OutPkts InBytes OutBytes
-> RemoteAddress:Port
TCP 192.168.112.10:80 113786 19495479 0 2110M 0
-> 192.168.112.13:80 12883 1679941 0 181747K 0
-> 192.168.112.14:80 40269 10218905 0 1105M 0
-> 192.168.112.15:80 40240 7403127 0 804275K 0
```

> –stats选项是统计自该条转发规则生效以来的
> Conns (connections scheduled) 已经转发过的连接数
> InPkts (incoming packets) 入包个数
> OutPkts (outgoing packets) 出包个数
> InBytes (incoming bytes) 入流量（字节）
> OutBytes (outgoing bytes) 出流量（字节）

- 修改 LVS 表中的 timeout

```
ipvsadm --set  900 60 300
ipvsadm -ln --timeout
Timeout (tcp tcpfin udp): 900 60 300
```