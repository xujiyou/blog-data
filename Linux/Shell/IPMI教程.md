## 安装 ipmitool

准备一台物理机，ipmi 是物理机上的一个硬件，它是独立于 CPU BIOS 和 OS 的，所以只要物理机是连接了电源的，即使操作系统是关闭的，也可以操作 ipmi。

安装 ipmitool：

```bash
$ yum install ipmitool
```

使用：

```bash
$ ipmitool
```

但是报错：

```
Could not open device at /dev/ipmi0 or /dev/ipmi/0 or /dev/ipmidev/0: No such file or directory
```

解决：

```
$ modprobe ipmi_devintf
$ modprobe ipmi_si
$ cat >> /etc/modules << EOF
ipmi_devintf
ipmi_si
EOF
```

但是在安装 `ipmi_si` 模块是报错：

```
modprobe: ERROR: could not insert 'ipmi_si': No such device
```

实际在系统中是有这个模块的：

```bash
$ ls /lib/modules/4.4.180-2.el7.elrepo.x86_64/kernel/drivers/char/ipmi/ipmi_si.ko
$ modinfo ipmi_si
```

为什么呐，因为 PC 上没有 IPMI 设备，服务器上才有 IPMI 设备。

IPMI 设备在 Linux 中对应的文件通常是 /dev/ipmi0 、 /dev/ipmi/0 、 /dev/ipmidev/0



## ipmitool 重置密码

查看用户列表：

```bash
$ sudo ipmitool user list
```

设置 admin 用户的密码：

```bash
$ ipmitool user set password 2 abcdefg
```



## ipmi 安装操作系统

在 Web 中打开 ipmi 的 IP 地址，输入用户名及密码。









