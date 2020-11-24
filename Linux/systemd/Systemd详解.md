# Systemd 详解

Systemd 官方文档：https://systemd.io/

在 CentOS 7 中，systemd 是进程号位为 1 的进程，它管理了 Linux 系统自开机以来的所有的进程。

Systemd目的是要取代Unix时代以来一直在使用的init系统，兼容SysV和LSB的启动脚本，而且够在进程启动过程中更有效地引导加载服务。

systemd的特性有：

- 支持并行化任务
- 同时采用socket式与D-Bus总线式激活服务；
- 按需启动守护进程（daemon）；
- 利用 Linux 的 cgroups 监视进程；
- 支持快照和系统恢复；
- 维护挂载点和自动挂载点；
- 各服务间基于依赖关系进行精密控制。



Systemd 被许多人记恨，原因就是不符合 Linux 精神，管的太多，甚至在某些方面借鉴 windows、osx 等闭源系统，比如 unit 配置文件就是 windows 的 ini 格式的。

## unit

systemd 开启和监督整个系统是基于 *unit* 的概念。*unit* 是由一个与配置文件对应的名字和类型组成的(例如：avahi.service *unit* 有一个具有相同名字的配置文件，是守护进程 Avahi 的一个封装单元)。*unit* 有以下几种类型：

1. `service` ：守护进程的启动、停止、重启和重载是此类 *unit* 中最为明显的几个类型。
2. `socket` ：此类 *unit* 封装系统和互联网中的一个 socket 。当下，systemd 支持流式、数据报和连续包的 AF_INET、AF_INET6、AF_UNIX socket 。也支持传统的 FIFOs 传输模式。每一个 socket *unit* 都有一个相应的服务 *unit* 。相应的服务在第一个“连接”进入 socket 或 FIFO 时就会启动(例如：nscd.socket 在有新连接后便启动 nscd.service)。
3. `device` ：此类 *unit* 封装一个存在于 Linux 设备树中的设备。每一个使用 udev 规则标记的设备都将会在 systemd 中作为一个设备 *unit* 出现。udev 的属性设置可以作为配置设备 *unit* 依赖关系的配置源。
4. `mount` ：此类 *unit* 封装系统结构层次中的一个挂载点。
5. `automount` ：此类 *unit* 封装系统结构层次中的一个自挂载点。每一个自挂载 *unit* 对应一个已挂载的挂载 *unit* (需要在自挂载目录可以存取的情况下尽早挂载)。
6. `target` ：此类 *unit* 为其他 *unit* 进行逻辑分组。它们本身实际上并不做什么，只是引用其他 *unit* 而已。这样便可以对 *unit* 做一个统一的控制。(例如：multi-user.target 相当于在传统使用 SysV 的系统中运行级别5)；bluetooth.target 只有在蓝牙适配器可用的情况下才调用与蓝牙相关的服务，如：bluetooth 守护进程、obex 守护进程等）
7. `snapshot` ：与 target *unit* 相似，快照本身不做什么，唯一的目的就是引用其他 *unit* 。



## 运行级别

Linux 运行级别是 sysvinit 中的概念，在 systemd 中，运行级别得到了替换。

运行级别如下：

- 0 停机，关机
- 1 单用户，无网络连接，不运行守护进程，不允许非超级用户登录
- 2 多用户，无网络连接，不运行守护进程
- 3 多用户，正常启动系统
- 4 用户自定义
- 5 多用户，带图形界面
- 6 重启

在 systemd 中，运行级别被 target 替换了：

```bash
$ ls -al /usr/lib/systemd/system/runlevel*.target
lrwxrwxrwx. 1 root root 15 Jul 21 22:57 /usr/lib/systemd/system/runlevel0.target -> poweroff.target
lrwxrwxrwx. 1 root root 13 Jul 21 22:57 /usr/lib/systemd/system/runlevel1.target -> rescue.target
lrwxrwxrwx. 1 root root 17 Jul 21 22:57 /usr/lib/systemd/system/runlevel2.target -> multi-user.target
lrwxrwxrwx. 1 root root 17 Jul 21 22:57 /usr/lib/systemd/system/runlevel3.target -> multi-user.target
lrwxrwxrwx. 1 root root 17 Jul 21 22:57 /usr/lib/systemd/system/runlevel4.target -> multi-user.target
lrwxrwxrwx. 1 root root 16 Jul 21 22:57 /usr/lib/systemd/system/runlevel5.target -> graphical.target
lrwxrwxrwx. 1 root root 13 Jul 21 22:57 /usr/lib/systemd/system/runlevel6.target -> reboot.target
```

默认级别：

```bash
$ ls -al /etc/systemd/system/default.target
lrwxrwxrwx. 1 root root 37 Sep 14 12:42 /etc/systemd/system/default.target -> /lib/systemd/system/multi-user.target
```

切换运行级别：

```bash
$ systemctl isolate runlevel3.target
```



## systemd-logind

Linux 系统的登录系统也被 systemd 控制了。`/usr/lib/systemd/system/systemd-logind.service` 是服务文件，`/usr/lib/systemd/systemd-logind` 是启动命令。

查看状态：

```bash
$ systemctl status systemd-logind
```



































