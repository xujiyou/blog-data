# 添加或去掉 OSD

官方教程：https://ceph.readthedocs.io/en/latest/rados/operations/add-or-rm-osds/

当群集启动并运行时，可以在运行时添加OSD或从群集中删除OSD。



## 添加 OSD

当您要扩展群集时，可以在运行时添加OSD。对于Ceph，OSD通常是主机中一个存储驱动器的一个Ceph ceph-osd守护程序。如果主机有多个存储驱动器，则可以为每个驱动器映射一个ceph-osd守护程序。

通常，最好检查群集的容量以查看是否达到其容量的上限。当群集达到接近满负荷的比率时，您应该添加一个或多个OSD以扩展群集的容量。

> 添加OSD之前，请勿让群集达到其全部容量。群集达到其接近满比率后发生的OSD故障可能会导致群集超过其满比率。



## 删除 OSD

当您要减小群集的大小或更换硬件时，可以在运行时删除OSD。

删除OSD之前，需要将其从群集中取出，以便Ceph可以开始重新平衡并将其数据复制到其他OSD：

```bash
$ ceph osd out {osd-num}
```



#### 观察数据迁移

将OSD移出群集后，Ceph将通过从您删除的OSD迁移放置组开始重新平衡群集。您可以使用ceph工具观察此过程。

```bash
$ ceph -w
```



#### 停止 OSD

将OSD从群集中取出后，它可能仍在运行。也就是说，OSD可能会弹出。从配置中删除OSD之前，必须先停止它。

```bash
$ sudo systemctl stop ceph-osd@{osd-num}
```

一旦停止OSD，它就会处于 down 状态。



#### 删除 OSD

此过程从群集映射中删除OSD，删除其身份验证密钥，从OSD映射中删除OSD，并从ceph.conf文件中删除OSD。如果主机有多个驱动器，则可能需要通过重复此过程为每个驱动器删除OSD。

1. 让群集首先忘记OSD。此步骤从CRUSH映射中删除OSD，并删除其身份验证密钥。并且它也从OSD映射中删除。

   ```bash
   $ ceph osd purge {id} --yes-i-really-mean-it
   ```

2. 在 ceph.conf 中删除相关的 OSD 的配置。

3. 删掉 相关的 lvm：

   ```bash
   $ lsblk
   $ sudo dmsetup remove ceph--f03d9403--7a7a--45e0--9e37--698a63f122d9-osd--block--ec92b89c--d8b6--4007--8c26--4ddef645efe6
   ```

   

   



















