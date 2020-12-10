# netstat 命令详解



## 选项

- -r, --route 显示路由

- -I, --interfaces=<Iface> 显示统计信息

  ```
  $ netstat -I=enp3s0f0
  $ netstat --interfaces=enp3s0f0
  Kernel Interface table
  Iface             MTU    RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
  enp3s0f0         1500 6109323856      0  24315 0      5793324591      0      0      0 BMRU
  ```

- -i, --interfaces 效果同上，只是显示所有网卡的

- -g, --groups 显示多播功能群组组员名单；

-  -s, --statistics 显示网络统计，类似SNMP

  ```
  $ netstat -s
  ```

- -M, --masquerade 显示伪装的网络连线；我当前的系统不支持 ip_masquerade

- -n, --numeric 显示数字，而不是显示名称

- --numeric-hosts    不显示主机名，显示ip

- --numeric-ports 不显示端口名，显示数字

- --numeric-users 不显示用户名，显示用户ID

- -N, --symbolic 显示网络硬件外围设备的符号连接名称；

- -e, --extend 显示更多信息，比如显示 User 和 Inode

  ```
  $ sudo netstat -nltpe
  Active Internet connections (only servers)
  Proto Recv-Q Send-Q Local Address           Foreign Address         State       User       Inode      PID/Program name    
  tcp        0      0 0.0.0.0:21100           0.0.0.0:*               LISTEN      101        59239      8564/nginx: master  
  tcp        0      0 0.0.0.0:21101           0.0.0.0:*               LISTEN      101        59241      8564/nginx: master  
  tcp        0      0 0.0.0.0:30030           0.0.0.0:*               LISTEN      0          19317      2657/kube-proxy     
  tcp        0      0 0.0.0.0:30031           0.0.0.0:*               LISTEN      0          19316      2657/kube-proxy 
  ```

- -p, --programs 显示某个PID的连接

- -o, --timers 显示计时器，和状态，通常配合 -p 使用。

  ```
  $ sudo netstat -pno 2655
  Active Internet connections (w/o servers)
  Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name     Timer
  tcp        0      0 10.42.0.0:44808         10.42.4.127:9300        TIME_WAIT   -                    timewait (19.62/0/0)
  tcp        0      0 10.28.112.11:80         10.28.111.9:39496       TIME_WAIT   -                    timewait (5.78/0/0)
  tcp        0      0 10.28.112.11:60868      10.28.112.11:2379       ESTABLISHED 3865/kube-apiserver  keepalive (10.87/0/0)
  tcp        0      0 10.28.112.11:6379       10.28.111.9:34884       ESTABLISHED 428918/nginx: worke  off (0.00/0/0)
  tcp        0      0 10.28.112.11:50988      10.28.112.11:6816       ESTABLISHED 1609/ceph-mgr        off (0.00/0/0)
  tcp        0      0 10.28.112.11:33520      10.28.112.11:2379       ESTABLISHED 3865/kube-apiserver  keepalive (10.94/0/0)
  tcp        0      0 10.28.112.11:38334      10.28.112.13:2379       ESTABLISHED 3865/kube-apiserver  keepalive (14.13/0/0)
  ```

-  -c, --continuous 持续列出网络状态；
- -l, --listening 只显示 LISTEN 状态的连接
- -a, --all 显示所有 connected 状态的连接
- -F, --fib 显示转发信息，默认选项
- -C, --cache
- -Z, --context 显示处于SELinux安全上下文的套接字





















