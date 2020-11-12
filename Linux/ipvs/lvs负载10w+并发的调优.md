# LVS 负载10w+并发的调优

### LVS参数调优

- 增大ipvs模块hash table的大小

ipvs模块hash table默认值为2^12=4096，改为2^20=1048576。可以用`ipvsadm -l`命令查询当前hash table的大小。

```
IP Virtual Server version 1.2.1 (size=4096)
```

修改方法：

在`/etc/modprobe.d/`目录下添加文件`ip_vs.conf`，内容为：

```
options ip_vs conn_tab_bits=20
```

重新加载ipvs模块。

```
IP Virtual Server version 1.2.1 (size=1048576)
```



### 9.2 文件句柄及进程数

```
*  soft nofile 1024000
*  hard nofile 1024000
*  soft nproc 1024000
*  hard nproc 1024000
```



### 9.3 内核参数

```properties
fs.file-max = 1048576
net.ipv4.ip_forward = 1
net.core.wmem_default = 8388608
net.core.wmem_max = 16777216
net.core.rmem_default = 8388608
net.core.rmem_max = 16777216
net.core.somaxconn = 65535
net.core.optmem_max = 81920
net.core.netdev_max_backlog = 262144
net.ipv4.route.gc_timeout = 20
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_abort_on_overflow = 1
net.ipv4.tcp_max_tw_buckets = 6000
net.ipv4.tcp_sack = 1
net.ipv4.tcp_window_scaling = 1
net.ipv4.tcp_no_metrics_save = 1
net.ipv4.tcp_rmem = 32768   131072  16777216
net.ipv4.tcp_wmem = 8192   131072  16777216
net.ipv4.tcp_mem = 94500000 915000000 927000000
net.ipv4.tcp_max_syn_backlog = 262144
net.ipv4.tcp_max_orphans = 3276800
net.ipv4.tcp_timestamps = 0
net.ipv4.tcp_synack_retries = 1
net.ipv4.tcp_syn_retries = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_fin_timeout = 10
net.ipv4.tcp_keepalive_time = 120
net.ipv4.tcp_keepalive_probes = 3
net.ipv4.tcp_keepalive_intvl = 15
net.ipv4.tcp_retries2 = 5
net.ipv4.ip_local_port_range = 1024    65000
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.conf.all.arp_ignore = 1
net.ipv4.conf.all.arp_announce = 2
#modprobe ip_conntrack 
net.netfilter.nf_conntrack_tcp_timeout_established = 180
net.netfilter.nf_conntrack_max = 1048576
net.nf_conntrack_max = 1048576
kernel.sysrq = 0
kernel.core_uses_pid = 1
kernel.msgmnb = 65536
kernel.msgmax = 65536
kernel.shmmax = 68719476736
kernel.shmall = 4294967296
```