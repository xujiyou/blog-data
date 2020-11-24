# journalctl 命令详解

当启动service失败或者出现异常时，我们同行需要查看systemd的日志。 journalctl就是最常用的查看systemd日志的工具了。

`journald` 的服务的 service 文件在：`/usr/lib/systemd/system/systemd-journald.service`

使用以下命令来查看 `journald` 的服务状态：

```bash
$ systemctl status systemd-journald
```

Journald 的日志是压缩和格式化的二进制数据。

存放运行日志的目录默认是(CentOS)：`/run/log/journal`

配置文件是 `/etc/systemd/journald.conf`

查看日志总占用量：

```bash
$ sudo journalctl --disk-usage
Archived and active journals take up 1.5G on disk.
$ sudo du -sh /run/log/journal
1.6G    /run/log/journal
```

可以看到两者查出来的是差不多的。

默认情况下并不会持久化保存日志，只会保留一个月的日志。如果需要永久保留改日志文件呢？

参考：https://yq.aliyun.com/articles/601749

```
$ sudo mkdir /var/log/journal
$ sudo chgrp systemd-journal /var/log/journal
$ sudo chmod g+s /var/log/journal
$ sudo systemctl restart systemd-journald
```

完成后，可以看到 `/var/log/journal` 中也有日志了。



## 常用操作

查看日志所占磁盘容量：

```
$ sudo journalctl --disk-usage
$ sudo journalctl -n 20
$ journalctl -u cron.service -n 3
```

只显示最新的 n 行

```bash
$ sudo journalctl -n 20
$ journalctl -u cron.service -n 3
```



## Flags

#### --system

显示系统日志

例子，显示最新的 10 条系统日志：

```bash
$ sudo journalctl --system -n 10
```



#### --user

显示当前用户的用户日志



#### -M --machine=CONTAINER

显示来自于正在运行的、特定名称的本地容器的日志。 参数必须是一个本地容器的名称。搞不太懂



#### -S --since=DATE

起始时间，例子：

```bash
$ sudo journalctl -S "2020-03-25 12:00:00" -n 10
```



#### -U --until=DATE

截止时间，例子：

```bash
$ sudo journalctl -S "2020-03-25 12:00:00" -U "2020-03-25 12:01:00"
```



#### -c --cursor=CURSOR

从指定的游标(cursor)开始显示日志。 [提示]每条日志都有一个"__CURSOR"字段，类似于该条日志的指纹。

例如：

```bash
$ sudo journalctl -n 1 -o json-pretty
{
        "__CURSOR" : "s=686b94637c39403cbe66f76075b90966;i=fd356d;b=9ca11a8943ce4d2e8ec141193c2231e2;m=17eaadd37fc;t=5a1a744f45738;x=f9fd3601e9abc4b5",
        "__REALTIME_TIMESTAMP" : "1585114672027448",
        "__MONOTONIC_TIMESTAMP" : "1643544131580",
        "_BOOT_ID" : "9ca11a8943ce4d2e8ec141193c2231e2",
        "PRIORITY" : "6",
        "_UID" : "0",
        "_GID" : "0",
        "_CAP_EFFECTIVE" : "3fffffffff",
        "_MACHINE_ID" : "134a37a1bf94838d52f64d0a78cb1e6e",
        "_HOSTNAME" : "fueltank-2.cloud.bbdops.com",
        "_TRANSPORT" : "syslog",
        "SYSLOG_FACILITY" : "10",
        "SYSLOG_IDENTIFIER" : "sudo",
        "_COMM" : "sudo",
        "_EXE" : "/usr/bin/sudo",
        "_AUDIT_SESSION" : "3430",
        "_AUDIT_LOGINUID" : "1003",
        "_SYSTEMD_CGROUP" : "/user.slice/user-1003.slice/session-3430.scope",
        "_SYSTEMD_SESSION" : "3430",
        "_SYSTEMD_OWNER_UID" : "1003",
        "_SYSTEMD_UNIT" : "session-3430.scope",
        "_SYSTEMD_SLICE" : "user-1003.slice",
        "MESSAGE" : "pam_unix(sudo:session): session opened for user root by admin(uid=0)",
        "_PID" : "20747",
        "_CMDLINE" : "sudo journalctl -n 1 -o json-pretty",
        "_SOURCE_REALTIME_TIMESTAMP" : "1585114672027030"
}
```

但是从网上好像搜不到这个参数的用法



#### --show-cursor

展示 cursor：

```bash
$ sudo journalctl -n 10 --show-cursor
```



#### -b --boot[=ID]

指定 boot id

```bash
$ sudo journalctl -n 1 -b 0
```



#### --list-boots

展示出 boot id。

```bash
$ sudo journalctl --list-boots
```



#### -k --dmesg

展示内核信息

```bash
$ sudo journalctl -n 10 -k
```



####  -u --unit=UNIT

显示哪个 unit 的日志：

```bash
$ sudo journalctl -n 10 -u kube-apiserver
```



#### -t --identifier=STRING

根据 syslog 的标识符来查询：

```bash
$ sudo journalctl -n 5 -t etcd
$ sudo journalctl -n 5 -t sudo
```



#### -p --priority=RANGE

根据日志的级别查询：

```
$ sudo journalctl -n 5 -p 7
$ sudo journalctl -n 5 -p 6
$ sudo journalctl -n 5 -p 5
$ sudo journalctl -n 5 -p 4
$ sudo journalctl -n 5 -p 3
```



#### -e --pager-end 

跳到结尾：

```bash
$ sudo journalctl -n 100 -e
```



#### -f --follow

在控制台上实时观看日志的输出：

```bash
$ sudo journalctl -n 100 -e -f
```



#### -n --lines[=INTEGER]

很常用的，指定查看的行数



#### --no-tail

显示所有行，即使在跟随模式下



#### -r --reverse

首先显示最新条目，默认就带上了



####  -o --output=STRING

输出模式，可选值有：(short, short-iso, short-precise, short-monotonic, verbose,export, json, json-pretty, json-sse, cat)



#### --utc

以（UTC）表示时间：

```bash
$ sudo journalctl -n 10 --utc
```



#### -x --catalog

在可用的地方添加消息说明

在日志的输出中 增加一些解释性的短文本， 以帮助进一步说明 日志的含义、 问题的解决方案、支持论坛、 开发文档、以及其他任何内容。 并非所有日志都有 这些额外的帮助文本， 详见 [Message Catalog Developer Documentation](https://www.freedesktop.org/wiki/Software/systemd/catalog) 文档。

```bash
$ sudo journalctl -n 3 -x
```



#### --no-full

省略超出屏幕的字符

```bash
$ sudo journalctl -n 3 --no-full
```



-a --all

打印所有字段：

```bash
$ sudo journalctl -n 3 -a
```



#### -q --quiet

不显示特权警告



#### --no-pager

打印完立即退出，而不是需要 ctrl + c 或 ZZ  或 q 才能退出

```bash
$ sudo journalctl -n 3 --no-pager
```



#### -m --merge 



#### -D --directory=PATH

展示某个目录中的日志

```bash
$ sudo journalctl -n 3 --no-pager -D ./
$ sudo journalctl -n 3 --no-pager -D /run/log/journal/
```



#### --file=PATH

展示某个文件中的日志

```bash
$ sudo journalctl -n 3 --no-pager --file=/run/log/journal/134a37a1bf94838d52f64d0a78cb1e6e/system.journal
```



#### --root=ROOT 

在根ROOT下的目录文件上操作



#### --interval=TIME

更改FSS密封密钥的时间间隔



#### --verify-key=KEY

指定FSS验证密钥



#### --force 

用--setup-keys覆盖FSS密钥对



## Commands



####  -h --help

显示帮助信息



#### --version 

显示版本信息



#### --new-id128

生成一个新的 128 位的 id



#### --disk-usage

查看磁盘日志的使用量



#### --flush

刷新日志从 /run 到 /var ，/run 是运行中的日志，是在内存中的，/var 是持久化在硬盘上的。



####  --header

显示每一个 journal 文件的元数据。



#### --list-catalog

Show all message IDs in the catalog



#### --dump-catalog

Show entries in the message catalog



#### --update-catalog

Update the message catalog database



#### --setup-keys

Generate a new FSS key pair



#### --verify

检查日志文件一致性。





