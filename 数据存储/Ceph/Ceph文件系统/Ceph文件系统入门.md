# Ceph 文件系统入门

Ceph 文件系统，名为 CephFS，兼容 POSIX，建立在 Ceph 对象储存 --- **RADOS**之上。



## 创建文件系统

首先创建两个 pool，一个存放文件系统的数据，一个储存元数据：

```bash
$ sudo ceph osd pool create cephfs_data
$ sudo ceph osd pool create cephfs_metadata
```

创建一个文件系统：

```bash
$ sudo ceph fs new cephfs cephfs_metadata cephfs_data
```

查看文件系统列表：

```bash
$ sudo ceph fs ls
```

查看文件系统状态：

```bash
$ sudo ceph mds stat
```



## 挂载文件系统

创建目录：

```bash
$ sudo mkdir /mnt/mycephfs
```

查看 admin 的key：

```bash
$ sudo cat ceph.client.admin.keyring
```

挂载：

```bash
$ sudo mount -t ceph 172.20.20.162:6789,172.20.20.179:6789,172.20.20.145:6789:/ /mnt/mycephfs -o name=admin,secret=AQBSLZVe4RjqKxAAsKLUdBsjDAlV7Ls06TeUfw==
```

查看效果：

```bash
$ df -h
```

取消挂载：

```bash
$ sudo umount /mnt/mycephfs
```





## Pool 创建错误记录

我这里在创建 pool 过程中，报了一个错：

```
Error ERANGE:  pg_num 32 size 3 would mean 819 total pgs, which exceeds max 750 (mon_max_pg_per_osd 250 * num_in_osds 3)
```

说下解决思路，就是为每个 pool 设置的 pg 数量太多了，而且 pool 也太多，导致达到了最大 pg 数量。这里解决方法有三个，第一是减少 pool 数量，把没有用的 pool 删掉，第二是缩小每个 pool 所拥有的 pg 数量，因为有些管理组件也会占用一个 pool，第三就是调大最大 pg 数量限制。

我这里选用了第三种方法，调大 pg 数量。过程如下：

先修改 ceph.conf 文件，将以下内容加入到 global 中

```
mon_max_pg_per_osd = 300
```

然后将新配置推送到各个节点：

```bash
$ sudo ceph-deploy --overwrite-conf config push fueltank-1 fueltank-2 fueltank-3
```

然后在各个节点分别重启：

```bash
$ sudo systemctl restart ceph-mon.target
$ sudo systemctl restart ceph-mgr.target
```

查看每个 osd 所拥有的最大 pg 数量：

```bash
$ sudo ceph --admin-daemon /var/run/ceph/ceph-mon.fueltank-1.asok config get  mon_max_pg_per_osd
```



## 文件系统启动不起来的错误

查看文件系统状态，发现失败：

```bash
$ sudo ceph mds stat
cephfs:0
```

查看错误原因：

```bash
$ sudo ceph health detail
```

响应如下：

```
HEALTH_ERR 1 filesystem is offline; 1 filesystem is online with fewer MDS than max_mds
[ERR] MDS_ALL_DOWN: 1 filesystem is offline
    fs cephfs is offline because no MDS is active for it.
[WRN] MDS_UP_LESS_THAN_MAX: 1 filesystem is online with fewer MDS than max_mds
    fs cephfs has 0 MDS online, but wants 1
```

这是因为文件系统需要启动 ceph-mds ！！！！！！

ceph-mds 用来管理文件系统的元数据，只有文件系统才需要，对象储存和块储存都不需要。 

部署：

```bash
$ sudo ceph-deploy mds create fueltank-1 fueltank-2 fueltank-3
```

部署完成后，就自动变好了。





