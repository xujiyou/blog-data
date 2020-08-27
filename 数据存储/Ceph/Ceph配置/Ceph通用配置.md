# Ceph 通用配置

官方教程：https://ceph.readthedocs.io/en/latest/rados/configuration/common/

一个宿主机可以运行多个守护程序，例如，具有多个磁盘的单个节点可以为每个磁盘运行一个ceph-osd。某些节点可能运行ceph-osd守护程序，其他节点可能运行ceph-mds守护程序，而另一些节点可能运行ceph-mon守护程序。

每个节点都有一个和主机名一样的名称。监视器还指定由addr设置标识的网络地址和端口（即域名或IP地址）。基本配置文件通常只会为每个监视器守护程序实例指定最少的设置。例如：

```ini
[global]
mon_initial_members = ceph1
mon_host = 10.0.0.1
```

> 主机名是节点的简称，即不是fqdn，也不是IP地址。在命令行上输入`hostname -s`以检索节点的名称。



## 网络

见 [Ceph网络配置.md](Ceph网络配置.md) 

## MONITORS

Ceph生产集群通常至少部署3个Ceph Monitor守护程序，以确保在监视器实例崩溃时具有高可用性。至少三 3 个监视器确保 Paxos 算法可以从仲裁中的大多数 Ceph Monitor 中确定哪个版本的 Ceph Cluster Map 是最新的。

> 您可以使用单个监视器部署Ceph，但是如果实例失败，则缺少其他监视器可能会中断数据服务的可用性。

Ceph Monitor 通常在端口 3300 上侦听新的 v2 协议，在端口 6789 上侦听旧的 v1 协议。

默认情况下，Ceph希望您将监视器的数据存储在以下路径下：

```
/var/lib/ceph/mon/$cluster-$id
```

必须创建相应的目录。使用集群名 （默认 Ceph）加上主机名（例如 a），则储存目录：

```
/var/lib/ceph/mon/ceph-a
```



## 认证

应该在Ceph配置文件的[global]部分中明确启用或禁用身份验证：

```ini
auth cluster required = cephx
auth service required = cephx
auth client required = cephx
```





## OSDS

Ceph生产集群通常部署Ceph OSD守护程序，其中有些节点有 OSD 守护程序，在一个存储驱动器上运行一个文件存储。典型部署指定日志大小。例如：

```ini
[osd]
osd journal size = 10000

[osd.0]
host = {hostname} #manual deployments only.
```

默认情况下，Ceph希望您使用以下路径存储Ceph OSD守护程序的数据：

```
/var/lib/ceph/osd/$cluster-$id
```

默认集群名是 ceph，对于第一个 OSD， id 为 0，则实际的储存目录为：

```
/var/lib/ceph/osd/ceph-0
```

您可以使用osd数据设置覆盖此路径。我们不建议更改默认位置。在OSD主机上创建默认目录：

```bash
ssh {osd-host}
sudo mkdir /var/lib/ceph/osd/ceph-{osd-number}
```

理想情况下，osd 数据路径通向硬盘的挂载点，该挂载点与存储和运行操作系统和守护程序的硬盘分开。如果OSD用于OS磁盘以外的其他磁盘，请准备将其与Ceph一起使用，并将其安装到刚创建的目录中：

```bash
ssh {new-osd-host}
sudo mkfs -t {fstype} /dev/{disk}
sudo mount -o user_xattr /dev/{hdd} /var/lib/ceph/osd/ceph-{osd-number}
```

我们建议在运行mkfs时使用xfs文件系统。 （不建议使用btrfs和ext4，并且不再对其进行测试。）





## HeartBeats

在运行时操作期间，Ceph OSD守护程序会检查其他Ceph OSD守护程序，并将其发现结果报告给Ceph Monitor。您不必提供任何设置。但是，如果您遇到网络延迟问题，则可能希望修改设置。



## Ceph.conf 示例

```ini
[global]
fsid = {cluster-id}
mon initial members = {hostname}[, {hostname}]
mon host = {ip-address}[, {ip-address}]

#All clusters have a front-side public network.
#If you have two NICs, you can configure a back side cluster 
#network for OSD object replication, heart beats, backfilling,
#recovery, etc.
public network = {network}[, {network}]
#cluster network = {network}[, {network}] 

#Clusters require authentication by default.
auth cluster required = cephx
auth service required = cephx
auth client required = cephx

#Choose reasonable numbers for your journals, number of replicas
#and placement groups.
osd journal size = {n}
osd pool default size = {n}  # Write an object n times.
osd pool default min size = {n} # Allow writing n copy in a degraded state.
osd pool default pg num = {n}
osd pool default pgp num = {n}

#Choose a reasonable crush leaf type.
#0 for a 1-node cluster.
#1 for a multi node cluster in a single rack
#2 for a multi node, multi chassis cluster with multiple hosts in a chassis
#3 for a multi node cluster with hosts across racks, etc.
osd crush chooseleaf type = {n}
```



## 全局配置

官方文档：https://ceph.readthedocs.io/en/latest/rados/configuration/general-config-ref/



`fsid`

- Description：集群 ID
- Type：UUID
- Required：No.
- Default：N/A. 部署工具会生成

`admin socket`

- Description

  套接字，用于在守护程序上执行管理命令，而不管 Ceph Monitor 是否已建立仲裁。

- Type：String

- Required：No

- Default：`/var/run/ceph/$cluster-$name.asok`

`pid file`

- Description

  mon，osd或mds将在其中写入其PID的文件。文件名叫做 `/var/run/$cluster/$type.$id.pid`，当守护程序正常停止时，将删除pid文件。如果该进程未守护进程（即使用-f或-d选项运行），则不会创建pid文件。

- Type：String

- Required：No

- Default：No

`chdir`

- Description

  一旦启动并运行，目录Ceph守护进程就会更改为这个活动目录。默认/目录推荐。

- Type：String

- Required：No

- Default：`/`

`max open files`

- Description

  如果设置，则在 Ceph 存储群集启动时，Ceph 会在操作系统级别设置最大打开文件数（即文件描述符的最大数量）。它有助于防止Ceph OSD 守护进程用尽文件描述符。

- Type：64-bit Integer

- Required：No

- Default：`0`

`fatal signal handlers`

- Description

  如果已设置，我们将为SEGV，ABRT，BUS，ILL，FPE，XCPU，XFSZ，SYS信号安装信号处理程序，以生成有用的日志消息。

- Type：Boolean

- Default：`true`





