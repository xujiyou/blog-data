# HTTP 高并发场景下的 TCP 参数优化



## 文件句柄

因为 Linux 系统为每个 TCP 建立连接时，都要创建一个 socket 句柄，每个 socket 句柄同时也是一个文件句柄。而系统对用户打开的文件句柄是有限制的，看到这里，也就理解了为什么在高并发时会出现 "too many open files"。

显示当前文件句柄数量：

```
ulimit -n
```

在 `/etc/security/limits.conf` 最后添加如下四行调整最大文件句柄数量和最大进程数量：

```
* soft nproc  65535
* hard nproc  65535
* soft nofile 65535
* hard nofile 65535
```

重启当前命令行生效

> 注意
> soft nofile （软限制）是指Linux在当前系统能够承受的范围内进一步限制用户同时打开的文件数
> hard nofile （硬限制）是根据系统硬件资源状况(主要是系统内存)计算出来的系统最多可同时打开的文件数量
> 通常软限制小于或等于硬限制。
> `*` 表示所有用户，也可以指定具体的用户名
> 所有进程打开的文件描述符数不能超过 /proc/sys/fs/file-max
> 单个进程打开的文件描述符数不能超过user limit中nofile的soft limit
> nofile的hard limit不能超过/proc/sys/fs/nr_open

上面的修改只是对用户的单一进程允许打开的最大文件数进行的设置。但是Linux系统对所有用户打开文件数也有限制（硬限制），一般情况下，不用修改此值。

```Bash
# 查看所有用户打开文件数的最大限制（此值与硬件配置有关）
cat /proc/sys/fs/file-max
```

但是，文件数再大，随着并发量的上升，也总有用完的时候。这时候再看 “文件都可以用「打开(open) –> 读写(write/read) –> 关闭(close)」模式来进行操作”这句话，就会想到，可以把已经用过的文件句柄释放掉，来减少资源的占用，这就又涉及到了资源重新利用和回收的问题。有时候查看端口有特别多的 `TIME_WAIT` 时，就是因为连接本身是关闭的，但资源还没有释放。



## 优化  TIME_WAIT

查看处于各种连接状态的连接数量：

```
$ netstat -an | awk '/tcp/ {print $6}' | sort | uniq -c
   1176 ESTABLISHED
    106 LISTEN
      1 SYN_RECV
     78 TIME_WAIT
```

状态描述：

- CLOSED：无连接是活动的或正在进行
- LISTEN：服务器在等待进入呼叫
- SYN_RECV：一个连接请求已经到达，等待确认
- SYN_SENT：应用已经开始，打开一个连接
- ESTABLISHED：正常数据传输状态
- FIN_WAIT1：应用说它已经完成
- FIN_WAIT2：另一边已同意释放
- ITMED_WAIT：等待所有分组死掉
- CLOSING：两边同时尝试关闭
- TIME_WAIT：另一边已初始化一个释放
- LAST_ACK：等待所有分组死掉

如发现系统存在大量TIME_WAIT状态的连接，通过调整内核参数解决：

```properties
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_fin_timeout = 30
```

- net.ipv4.tcp_syncookies = 1 表示开启SYN cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭；
- net.ipv4.tcp_tw_reuse = 1 表示开启重用。允许将TIME_WAIT sockets重新用于新的TCP连接，默认为0，表示关闭；
- net.ipv4.tcp_tw_recycle = 1 表示开启TCP连接中TIME_WAIT sockets的快速回收，默认为0，表示关闭。
- net.ipv4.tcp_fin_timeout 修改系統默认的 TIMEOUT 时间



#### TIME_WAIT状态的意义
客户端与服务器端建立TCP/IP连接后关闭SOCKET后，服务器端连接的端口状态为TIME_WAIT。

**是不是所有执行主动关闭的socket都会进入TIME_WAIT状态呢？**

**有没有什么情况使主动关闭的socket直接进入CLOSED状态呢？**

主动关闭的一方在发送最后一个 ack 后就会进入 TIME_WAIT 状态 停留2MSL（max segment lifetime）时间

这个是TCP/IP必不可少的，也就是“解决”不了的。也就是TCP/IP设计者本来是这么设计的

主要有两个原因：

1. 防止上一次连接中的包，迷路后重新出现，影响新连接（经过2MSL，上一次连接中所有的重复包都会消失）
2. 可靠的关闭TCP连接

在主动关闭方发送的最后一个 ack(fin) ，有可能丢失，这时被动方会重新发fin, 如果这时主动方处于 CLOSED 状态 ，就会响应 rst 而不是 ack。所以主动方要处于 TIME_WAIT 状态，而不能是 CLOSED 。

TIME_WAIT 并不会占用很大资源的，除非受到攻击。

还有，如果一方 send 或 recv 超时，就会直接进入 CLOSED 状态



## TCP 参数调优

网上有很多关于 `tcp_tw_reuse` 和 `tcp_tw_recycle` 的设置，但是很多没有说明场景，有时修改并不一定有好处。详细说明请看 [不要在linux上启用net.ipv4.tcp_tw_recycle参数](http://www.cnxct.com/coping-with-the-tcp-time_wait-state-on-busy-linux-servers-in-chinese-and-dont-enable-tcp_tw_recycle/) 和 [tcp_tw_reuse、tcp_tw_recycle 使用场景及注意事项](http://www.cnblogs.com/lulu/p/4149312.html) 。

所以，以上两个参数，在不是十分确定的场景情况下，不建议修改开启。

可以通过以下几个参数进行优化。

| 参数                          | 默认配置             | 调整配置              | 说明                                                         |
| :---------------------------- | :------------------- | :-------------------- | :----------------------------------------------------------- |
| fs.file-max                   | 1048576              | 9999999               | 所有进程打开的文件描述符数                                   |
| fs.nr_open                    | 1635590              | 1635590               | 单个进程可分配的最大文件数                                   |
| net.core.rmem_default         | 124928               | 262144                | 默认的TCP读取缓冲区                                          |
| net.core.wmem_default         | 124928               | 262144                | 默认的TCP发送缓冲区                                          |
| net.core.rmem_max             | 124928               | 8388608               | 默认的TCP最大读取缓冲区                                      |
| net.core.wmem_max             | 124928               | 8388608               | 默认的TCP最大发送缓冲区                                      |
| net.ipv4.tcp_wmem             | 4096 16384 4194304   | 4096 16384 8388608    | TCP发送缓冲区                                                |
| net.ipv4.tcp_rmem             | 4096 87380 4194304   | 4096 87380 8388608    | TCP读取缓冲区                                                |
| net.ipv4.tcp_mem              | 384657 512877 769314 | 384657 512877 3057792 | TCP内存大小                                                  |
| net.core.netdev_max_backlog   | 1000                 | 5000                  | 在每个网络接口接收数据包的速率比内核处理这些包的速率快时，允许送到队列的数据包的最大数目 |
| net.core.optmem_max           | 20480                | 81920                 | 每个套接字所允许的最大缓冲区的大小                           |
| net.core.somaxconn            | 128                  | 2048                  | 每一个端口最大的监听队列的长度，这是个全局的参数             |
| net.ipv4.tcp_fin_timeout      | 60                   | 30                    | 对于本端断开的socket连接，TCP保持在FIN-WAIT-2状态的时间（秒）。对方可能会断开连接或一直不结束连接或不可预料的进程死亡 |
| net.core.netdev_max_backlog   | 1000                 | 10000                 | 在每个网络接口接收数据包的速率比内核处理这些包的速率快时，允许送到队列的数据包的最大数目 |
| net.ipv4.tcp_max_syn_backlog  | 1024                 | 2048                  | 对于还未获得对方确认的连接请求，可保存在队列中的最大数目。如果服务器经常出现过载，可以尝试增加这个数字 |
| net.ipv4.tcp_max_tw_buckets   | 5000                 | 5000                  | 系统在同时所处理的最大timewait sockets数目                   |
| net.ipv4.tcp_tw_reuse         | 0                    | 1                     | 是否允许将TIME-WAIT sockets重新用于新的TCP连接               |
| net.ipv4.tcp_keepalive_time   | 7200                 | 900                   | 表示TCP链接在多少秒之后没有数据报文传输时启动探测报文（发送空的报文） |
| net.ipv4.tcp_keepalive_intvl  | 75                   | 30                    | 表示前一个探测报文和后一个探测报文之间的时间间隔             |
| net.ipv4.tcp_keepalive_probes | 9                    | 3                     | 表示探测的次数                                               |

从以上参数可以看出，Linux 内核为 TCP 发送和接收做了缓冲队列，以提高其吞吐量。

以上这些参数都是在 `/etc/sysctl.conf` 文件中定义的，有的参数在文件中可能没有定义，系统给定了默认值，需要修改的话，直接在文件中添加或修改，然后执行`sysctl -p`命令让其生效。

> 注意
> 参数值并不是设置的越大越好，有的需要考虑服务器的硬件配置，参数对服务器上其它服务的影响等。



## 本地端口号

有时候我们修改了文件句柄限制数后，错误日志又会提示 "Can’t assignrequested address"。这是因为TCP 建立连接，在创建 Socket 句柄时，需要占用一个本地端口号（与 TCP 协议端口号不一样），相当于一个进程，便于与其它进程进行交互。而Linux内核的TCP/IP 协议实现模块对本地端口号的范围进行了限制。当端口号用尽，就会出现这种错误了。

我们可以修改本地端口号的范围。

```Bash
# 查看IP协议本地端口号限制
cat /proc/sys/net/ipv4/ip_local_port_range

#一般系统默认为以下值
32768    61000

#修改本地端口号
vim /etc/sysctl.conf

#修改参数
net.ipv4.ip_local_port_range = 1024 65000

#保存修改后，需要执行sysctl命令让修改生效
sysctl -p
```

> **注意：**
> 1、net.ipv4.ip_local_port_range的最小值为1024，1024以下的端口已经规划为TCP协议占用，如果想将 TCP 协议端口设置为8080等大端口号，可以将这里的最小值调大。
>
> 2、如果存在应用服务端口号大于1024的，应该将 net.ipv4.ip_local_port_range 的起始值修改为大于应用服务端口号，否则服务会报错。

是不是设置了这些参数之后，并发性能就显著提升呢？答案是不一定。还需要看服务是否使用了长连接。





## 调优实践

[lvs负载10w+并发的调优.md](../../../Linux/ipvs/lvs负载10w+并发的调优.md) 















