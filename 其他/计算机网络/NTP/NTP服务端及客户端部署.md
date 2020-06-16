# NTP 服务端及客户端部署

NTP是网络时间协议(Network Time Protocol)，它是用来同步网络中各个计算机的时间的协议。

教程：http://codeshold.me/2016/11/ntp-server.html



## 服务端安装

```bash
$ sudo yum install -y ntp
```

配置 文件为 `/etc/ntp.conf` 

```
restrict default nomodify notrap nopeer noquery
```

第一行restrict允许其他客户端请求你的时间服务器，**保持默认即可**(默认是未注释的)！
详细参数说明如下：

- noquery 禁止从该ntpd中dump状态数据
- notrap 禁止控制消息trap该服务器
- nomodify 禁止所有企图修改该服务器的ntpq请求
- nopeer 禁止所有企图建立peer association的数据包

ntp.conf中添加：

```
restrict 172.20.20.224 mask 255.255.252.0 nomodify notrap
```

只允许这个网段的连接。

如果本地主机需要查询和修改权限，再添加`restrict 127.0.0.1` –> **建议添加**，若有保持默认即可!

如果服务器不能联网，可以将以下内容注释掉

```
#server 0.centos.pool.ntp.org iburst
#server 1.centos.pool.ntp.org iburst
#server 2.centos.pool.ntp.org iburst
#server 3.centos.pool.ntp.org iburst
```



### 将本地时间作为备份

在ntp.conf中添加如下内容，注意是**127.127.1.0**!!! 必须是这个。

```
server 127.127.1.0 # 本机时间
fudge 127.127.1.0 stratum 10
```



我的 server 端的全部配置：

```
driftfile /var/lib/ntp/drift

restrict default nomodify notrap nopeer noquery

restrict 127.0.0.1
restrict ::1
restrict 172.20.20.224 mask 255.255.252.0 nomodify notrap

server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst
server 127.127.1.0 # 本机时间
fudge 127.127.1.0 stratum 10


includefile /etc/ntp/crypto/pw

keys /etc/ntp/keys

disable monitor
```



###  启动ntp服务器

```bash
$ sudo systemctl start ntpd
$ sudo systemctl enable ntpd
```

查看状态：

```bash
$ sudo systemctl status ntpd
$ ntpq -p
$ ntpstat
```



### 验证 ntp

```bash
$ sudo ntpdate -u drift-1
```

drift-1 是上面 ntp 服务器的主机名。



## 客户端安装

### ntpdate和ntpd进行时间同步的区别

ntpdate只能用来将本机时间与服务器进行同步，而ntpd还可以让别的机器将其作为时间同步服务器进行时间同步。

ntpdate是一下将本机时间修改成与服务器时间相一致，对采用了系统时间的程序而言是不友好的；ntpd是多次缓慢地对时间进行调整使本地时间与服务器时间相一致，是更推荐的。

若采用 ntpdate ，可以配合定时任务来完成。



若采用 ntpd 作为 客户端，配置文件如下：

```
driftfile /var/lib/ntp/drift
restrict default nomodify notrap nopeer noquery
restrict 127.0.0.1
restrict ::1
server 172.20.20.224 prefer
includefile /etc/ntp/crypto/pw
keys /etc/ntp/keys
disable monitor
```

其他步骤和服务端一致。

完成同步需要几分钟的时间。

查看时间是否已经同步了：

```bash
$ timedatectl
      Local time: Tue 2020-06-16 09:45:29 CST
  Universal time: Tue 2020-06-16 01:45:29 UTC
        RTC time: Tue 2020-06-16 01:45:28
       Time zone: Asia/Shanghai (CST, +0800)
     NTP enabled: yes
NTP synchronized: yes
 RTC in local TZ: no
      DST active: n/a
```

查看同步状态：

```
$ ntpstat
synchronised to NTP server (172.20.20.224) at stratum 4
   time correct to within 1079 ms
   polling server every 64 s
```

或者：

```
$ ntpq -p
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
*drift-1         51.75.17.219     3 u   53   64   37    0.392   -4.074   6.987
```









