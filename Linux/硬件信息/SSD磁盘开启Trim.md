# SSD 磁盘开启 Trim

TRIM 功能使操作系统得以通知 SSD 哪些页不再包含有效的数据。TRIM 功能有助于延长 SSD 的长期性能和使用寿命。如果要启用 TRIM, 需要确认 SSD 、操作系统、文件系统都支持 TRIM。

固态硬盘（或固态硬盘），能够实现更快的读取和相比传统硬盘数据的写入速度。但是你可能不知道的是，随着时间的推移，当磁盘写满时，SSD可能会失去某些速度。如果出于速度考虑在服务器中运行SSD，请按照以下方法使用TRIM使SSD保持最佳状态。

## **为什么SSD会变慢？**

首先让我们看看为什么会出现此问题。这与SSD的写入方式有关[数据](https://www.100tb.com/your-industry/big-data/)到存储。SSD将数据存储在固定大小的块（称为页面）中。然后将这些页面按称为块的较大组进行排列。尽管SSD可以单独读取和写入页面，但它们只能擦除数据块，而不能擦除单个页面。与可以覆盖数据块没有任何问题的硬盘驱动器不同，SSD需要先擦除块中的数据，然后才能将新数据写入内部页面。这在设计操作系统和文件系统时就成为问题，如果删除了文件，则将使用的文件块标记为可写入文件系统，但是这些块中的数据将保留到新数据为止。写在顶部。这是取消删除和文件恢复工具利用此原理从磁盘中恢复已删除文件的原理。





## 检测 SSD 是否支持 TRIM

可以通过 /sys/block 下的信息来判断 SSD 支持 TRIM, discard_granularity 非 0 表示支持。

```bash
# cat /sys/block/sda/queue/discard_granularity
0
# cat /sys/block/nvme0n1/queue/discard_granularity
512
```

也可以直接使用 lsblk 来检测，DISC-GRAN (discard granularity) 和 DISC-MAX (discard max bytes) 列非 0 表示该 SSD 支持 TRIM 功能。

```bash
# lsblk --discard
NAME    DISC-ALN DISC-GRAN DISC-MAX DISC-ZERO
sda            0        0B       0B         0
├─sda1         0        0B       0B         0
├─sda2         0        0B       0B         0
└─sda3         0        0B       0B         0
sr0            0        0B       0B         0
nvme0n1      512      512B       2T         1
nvme1n1      512      512B       2T         1
```

网上也有文章介绍通过 hdparm 来检测，不过我在 Intel P4500 SSD 测试没有返回该信息。

```bash
$ sudo yum install hdparm -y
$ sudo hdparm -I /dev/sda | grep TRIM
        *    Data Set Management TRIM supported (limit 1 block)
```



## 启用 TRIM

参考：https://zhuanlan.zhihu.com/p/34683444

TRIM是内置于ATA命令中的SSD命令，它是磁盘与计算机交互方式的一部分。操作系统能够将TRIM命令发送到磁盘，以使其知道哪些块是已删除文件的一部分，并允许SSD在需要写入之前先擦除这些块。虽然操作系统能够在每次驱动器删除文件系统上的文件时向驱动器发送信号以擦除这些部分，但这也会影响性能并减慢运行速度。因此，建议按计划运行TRIM以间歇性地清除块。

修改 `/etc/fstab` 中的记录，加入 discard 选项。

```
/dev/sdb1  /data1       xfs   defaults,noatime,discard   0  0
```

这里 ext4 和 xfs 都可以。

xfs 的更多挂载选项请见：https://www.kernel.org/doc/html/latest/admin-guide/xfs.html?highlight=fstrim

或者手动把磁盘 trim 一遍：

```bash
$ fstrim -a -v
```

-a标志告诉fstrim检查所有可用的有效分区，-v标志提供详细的输出，向您显示fstrim已完成的操作。您应该看到命令的输出，以查看运行情况，如果输出为正，则可以将命令添加到cron条目中。



## 定期执行

在 CentOS 7 中，已经自带了 `/usr/lib/systemd/system/fstrim.timer` 和 `/usr/lib/systemd/system/fstrim.service`

/usr/lib/systemd/system/fstrim.timer：

```
[Unit]
Description=Discard unused blocks once a week
Documentation=man:fstrim

[Timer]
OnCalendar=weekly
AccuracySec=1h
Persistent=true

[Install]
WantedBy=multi-user.target
```

/usr/lib/systemd/system/fstrim.service：

```
[Unit]
Description=Discard unused blocks

[Service]
Type=oneshot
ExecStart=/usr/sbin/fstrim -a
```

只需开启即可：

```bash
$ sudo systemctl enable fstrim.timer
$ sudo systemctl start fstrim.timer
$ sudo systemctl status fstrim.timer
```























