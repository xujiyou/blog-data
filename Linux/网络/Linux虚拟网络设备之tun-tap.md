---
title: Linux虚拟网络设备之tun/tap
date: 2020-07-02 20:12:04
tags:

---

云计算或者云原生技术，它们的网络管理都离不开虚拟网络设备，所以了解虚拟网络设备有利于我们更好的理解云时代的网络结构。

在使用 OpenStack 的时候，遇到了 tap 设备，这一节来详细了解下 tun/tap 是什么设备。



## 虚拟设备

对于一个网络设备来说，就像一个管道（pipe）一样，有两端，从其中任意一端收到的数据将从另一端发送出去。

比如一个物理网卡eth0，它的两端分别是内核协议栈（通过内核网络设备管理模块间接的通信）和外面的物理网络，从物理网络收到的数据，会转发给内核协议栈，而应用程序从协议栈发过来的数据将会通过物理网络发送出去。

那么对于一个虚拟网络设备呢？首先它也归内核的网络设备管理子系统管理，对于Linux内核网络设备管理模块来说，虚拟设备和物理设备没有区别，都是网络设备，都能配置IP，从网络设备来的数据，都会转发给协议栈，协议栈过来的数据，也会交由网络设备发送出去，至于是怎么发送出去的，发到哪里去，那是设备驱动的事情，跟Linux内核就没关系了，所以说虚拟网络设备的一端也是协议栈，而另一端是什么取决于虚拟网络设备的驱动实现。



## tun/tap的另一端是什么？

```
+----------------------------------------------------------------+
|                                                                |
|  +--------------------+      +--------------------+            |
|  | User Application A |      | User Application B |<-----+     |
|  +--------------------+      +--------------------+      |     |
|               | 1                    | 5                 |     |
|...............|......................|...................|.....|
|               ↓                      ↓                   |     |
|         +----------+           +----------+              |     |
|         | socket A |           | socket B |              |     |
|         +----------+           +----------+              |     |
|                 | 2               | 6                    |     |
|.................|.................|......................|.....|
|                 ↓                 ↓                      |     |
|             +------------------------+                 4 |     |
|             | Newwork Protocol Stack |                   |     |
|             +------------------------+                   |     |
|                | 7                 | 3                   |     |
|................|...................|.....................|.....|
|                ↓                   ↓                     |     |
|        +----------------+    +----------------+          |     |
|        |      eth0      |    |      tun0      |          |     |
|        +----------------+    +----------------+          |     |
|    10.32.0.11  |                   |   192.168.3.11      |     |
|                | 8                 +---------------------+     |
|                |                                               |
+----------------|-----------------------------------------------+
                 ↓
         Physical Network
```

上图中有两个应用程序A和B，都在用户层，而其它的socket、协议栈（Newwork Protocol Stack）和网络设备（eth0和tun0）部分都在内核层，其实socket是协议栈的一部分，这里分开来的目的是为了看的更直观。

tun0是一个Tun/Tap虚拟设备，从上图中可以看出它和物理设备eth0的差别，它们的一端虽然都连着协议栈，但另一端不一样，eth0的另一端是物理网络，这个物理网络可能就是一个交换机，而tun0的另一端是一个用户层的程序，协议栈发给tun0的数据包能被这个应用程序读取到，并且应用程序能直接向tun0写数据。

这里假设eth0配置的IP是10.32.0.11，而tun0配置的IP是192.168.3.11.

这里列举的是一个典型的tun/tap设备的应用场景，发到192.168.3.0/24网络的数据通过程序B这个隧道，利用10.32.0.11发到远端网络的10.33.0.1，再由10.33.0.1转发给相应的设备，从而实现VPN。

下面来看看数据包的流程：

应用程序A是一个普通的程序，通过socket A发送了一个数据包，假设这个数据包的目的IP地址是192.168.3.1

socket将这个数据包丢给协议栈

协议栈根据数据包的目的IP地址，匹配本地路由规则，知道这个数据包应该由tun0出去，于是将数据包交给tun0

tun0收到数据包之后，发现另一端被进程B打开了，于是将数据包丢给了进程B

进程B收到数据包之后，做一些跟业务相关的处理，然后构造一个新的数据包，将原来的数据包嵌入在新的数据包中，最后通过socket B将数据包转发出去，这时候新数据包的源地址变成了eth0的地址，而目的IP地址变成了一个其它的地址，比如是10.33.0.1.

socket B将数据包丢给协议栈

协议栈根据本地路由，发现这个数据包应该要通过eth0发送出去，于是将数据包交给eth0

eth0通过物理网络将数据包发送出去

10.33.0.1收到数据包之后，会打开数据包，读取里面的原始数据包，并转发给本地的192.168.3.1，然后等收到192.168.3.1的应答后，再构造新的应答包，并将原始应答包封装在里面，再由原路径返回给应用程序B，应用程序B取出里面的原始应答包，最后返回给应用程序A

这里不讨论Tun/Tap设备tun0是怎么和用户层的进程B进行通信的，对于Linux内核来说，有很多种办法来让内核空间和用户空间的进程交换数据。

从上面的流程中可以看出，数据包选择走哪个网络设备完全由路由表控制，所以如果我们想让某些网络流量走应用程序B的转发流程，就需要配置路由表让这部分数据走tun0。



## tun/tap设备有什么用？

从上面介绍过的流程可以看出来，tun/tap设备的用处是将协议栈中的部分数据包转发给用户空间的应用程序，给用户空间的程序一个处理数据包的机会。于是比较常用的数据压缩，加密等功能就可以在应用程序B里面做进去，tun/tap设备最常用的场景是VPN，包括tunnel以及应用层的IPSec等，比较有名的项目是VTun，有兴趣可以去了解一下。



## tun和tap的区别

用户层程序通过tun设备只能读写IP数据包，而通过tap设备能读写链路层数据包，类似于普通socket和raw socket的差别一样，处理数据包的格式不一样。



## 实例

写了一个程序，它收到tun设备的数据包之后，只打印出收到了多少字节的数据包，其它的什么都不做。

```c
#include <net/if.h>
#include <sys/ioctl.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <string.h>
#include <sys/types.h>
#include <linux/if_tun.h>
#include <stdlib.h>
#include <stdio.h>

int tun_alloc(int flags)
{

    struct ifreq ifr;
    int fd, err;
    char *clonedev = "/dev/net/tun";

    if ((fd = open(clonedev, O_RDWR)) < 0) {
        return fd;
    }

    memset(&ifr, 0, sizeof(ifr));
    ifr.ifr_flags = flags;

    if ((err = ioctl(fd, TUNSETIFF, (void *) &ifr)) < 0) {
        close(fd);
        return err;
    }

    printf("Open tun/tap device: %s for reading...\n", ifr.ifr_name);

    return fd;
}

int main()
{

    int tun_fd, nread;
    char buffer[1500];

    /* Flags: IFF_TUN   - TUN device (no Ethernet headers)
     *        IFF_TAP   - TAP device
     *        IFF_NO_PI - Do not provide packet information
     */
    tun_fd = tun_alloc(IFF_TUN | IFF_NO_PI);

    if (tun_fd < 0) {
        perror("Allocating interface");
        exit(1);
    }

    while (1) {
        nread = read(tun_fd, buffer, sizeof(buffer));
        if (nread < 0) {
            perror("Reading from interface");
            close(tun_fd);
            exit(1);
        }

        printf("Read %d bytes from tun/tap device\n", nread);
    }
    return 0;
}
```

编译：

```bash
$ gcc tun.c -o tun
```

启动程序：

```bash
$ ./tun
```

启动起来之后，程序会阻塞，在第二个 shell 窗口查看网卡，会发现多了一个 tun0：

```bash
$ ip a
```

为 tun0 添加新ip：

```bash
$ ip addr add 192.168.3.11/24 dev tun0
```

添加 IP 后，也会在主机上添加一条新的路由：

```
192.168.3.0/24 dev tun0 proto kernel scope link src 192.168.3.11
```



将 tun0 启动起来：

```bash
$ ip link set tun0 up
```

使用抓包程序，查看 tun0 的数据包：

```bash
$ tcpdump -i tun0
```

在第三个 shell 中 ping tun0 网段的一个 IP 地址：

```bash
$ ping -c 4 192.168.3.12
```

虽然 ping 程序没有收到包，但是在 tun 程序中，已经收到了包，并获取了数据包的字节数。



## 使用命令创建 tun/tap 设备



#### tunctl 

安装 `tunctl` ：

```bash
$ vim /etc/yum.repos.d/nux-misc.repo
```

内容如下：

```
[nux-misc]
name=Nux Misc
baseurl=http://li.nux.ro/download/nux/misc/el7/x86_64/
enabled=1
gpgcheck=1
gpgkey=http://li.nux.ro/download/nux/RPM-GPG-KEY-nux.ro
```

安装：

```bash
$ yum install tunctl
```

默认创建一个名为 tap0 的接口：

```bash
$ tunctl
```

添加 IP 地址：

```bash
$ ip addr add 192.168.3.11/24 dev tap0
$ ip link set tap0 up
```

删除 接口：

```bash
$ tunctl -d tap0
```



#### ip tuntap

创建 tap/tun 设备：

```bash
$ ip tuntap add dev tap0 mod tap # 创建 tap 
$ ip tuntap add dev tun0 mod tun # 创建 tun
```

删除 tap/tun 设备：

```bash
$ ip tuntap del dev tap0 mod tap # 删除 tap 
$ ip tuntap del dev tun0 mod tun # 删除 tun
```









