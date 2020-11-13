# SSH 反向代理与端口转发配置

## sshd 服务端设置

sshd 需要开启 X11-forwarding

1. Ensure `xorg-x11-xauth` is installed
2. /etc/ssh/sshd_config：

```
# AddressFamily any
AddressFamily inet

AllowTcpForwarding yes
X11Forwarding yes
X11DisplayOffset 10
X11UseLocalhost yes
```



## 端口转发



客户的6台机器处于内网，之前只能用向日葵访问，但是客户那里有一台处于公网的机器，可以用来做 SSH 反向代理。

公网机器信息：

```properties
//转发服务器
ip:123.123.123.123
os:CentOS 7.5
port:22
```

内网机器信息：

```properties
ip:192.168.225.2
port:22
```

首先，让内网机器可以免密登录公网

在内网机器操作：

```bash
ssh-copy-id root@123.123.123.123
```

然后在内网机器上使用 SSH 开启反向代理端口：

```bash
ssh -fCNR 10022:localhost:22 root@123.123.123.123
```

SSH 参数解释：

```
-f 后台执行ssh指令
-C 允许压缩数据
-N 不执行远程指令
-R 将远程主机(服务器)的某个端口转发到本地端指定机器的指定端口
-L 将本地机(客户机)的某个端口转发到远端指定机器的指定端口
-p 指定远程主机的端口
```

然后在公网机器上使用 SSH 开启正向代理端口：

````bash
ssh -fCNL *:20022:localhost:10022 root@localhost
````

公网机器防火墙需要放行 20022 端口，然后在自己电脑上就可以使用ssh连接内网的机器了：

```bash
ssh -p 20022 root@123.123.123.123
```



## 稳定版

不幸的是这种ssh反向链接会因为超时而关闭，如果关闭了那从外网连通内网的通道就无法维持了，为此我们需要另外的方法来提供稳定的ssh反向代理隧道。

安装 autossh：

```bash
yum install autossh
```

然后在内网机器上执行（为了实现开机启动，可以将下面的命令写入 `/etc/rc.d/rc.local`）：

```bash
autossh -M 10021 -fCNR 10022:localhost:22 root@123.123.123.123
```

`autossh`的参数与ssh的参数是一致的，但是不同的是，在隧道断开的时候，autossh会自动重新连接而ssh不会。另外不同的是我们需要指出的-M参数，这个参数指定一个端口，这个端口是外网的B机器用来接收内网A机器的信息，如果隧道不正常而返回给A机器让他实现重新连接。

查看 autossh 是否在运行：

```bash
ps aux | grep autossh
```

在公网机器上执行（为了实现开机启动，可以将下面的命令写入 `/etc/rc.d/rc.local`）：

```bash
ssh -fCNL *:20022:localhost:10022 root@localhost
```

因为公网机器上只是使用 ssh 做了一个端口映射，所以不存在因为网络不稳而断开的问题，所以不用使用 autossh。

这样之后，就再也不用手动操作了，使用下面的命令连内网的机器：

```bash
ssh -p 20022 root@123.123.123.123
```











