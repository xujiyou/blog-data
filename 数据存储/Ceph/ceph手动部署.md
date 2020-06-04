# ceph 手动部署

为了能够完整了解 Ceph 集群部署过程，我这里不采用 ceph-deploy 和 cephadm，而是采用手工通过 RPM 包管理工具进行部署。

我的 `/etc/yum.repos.d/ceph.repo` 如下：

```
[ceph]
name=Ceph packages
baseurl=http://mirrors.163.com/ceph/rpm-15.2.1/el7/x86_64/
enabled=1
gpgcheck=1
priority=2
type=rpm-md
gpgkey=http://mirrors.163.com/ceph/keys/release.asc

[ceph-noarch]
name=Ceph noarch packages
baseurl=http://mirrors.163.com/ceph/rpm-15.2.1/el7/noarch/
enabled=1
gpgcheck=1
priority=2
type=rpm-md
gpgkey=http://mirrors.163.com/ceph/keys/release.asc

[ceph-source]
name=Ceph source packages
baseurl=http://mirrors.163.com/ceph/rpm-15.2.1/el7/SRPMS/
enabled=0
gpgcheck=1
priority=2
type=rpm-md
gpgkey=http://mirrors.163.com/ceph/keys/release.asc
```

安装软件：

```bash
$ sudo yum install ceph 
```

会安装以下软件：

![image-20200528141052464](../../resource/image-20200528141052464.png)

这里面没有对象网关，需单独安装：

```bash
$ yum -y install ceph-radosgw
```



## 启动 ceph-mon

编写配置文件：

```bash
$ sudo vim /etc/ceph/ceph.conf
```

内容如下：

```
[global]
public network = 172.20.23.0/24
cluster network = 172.20.23.0/24
fsid = 2b19a4b1-cf7d-4eef-8e4d-eed63734d56d
mon initial members = fueltank-1,fueltank-2,fueltank-3
mon host = 172.20.20.162,172.20.20.179,172.20.20.145
auth_cluster_required = cephx
auth_service_required = cephx
auth_client_required = cephx
osd journal size = 10240
osd pool default size = 3
osd pool default min size = 1
osd pool default pg num = 64
osd pool default pgp num = 64
osd crush chooseleaf type = 1
osd_mkfs_type = xfs
max mds = 5
mds max file size = 100000000000000
mds cache size = 1000000
mon osd down out interval = 900

[mon]
mon clock drift allowed = .50
mon allow pool delete = true
```

其中 `fsid` 是使用命令 `uuidgen` 生成的。

我的当前目录为 `/home/admin/ceph/new` ，为集群创建密钥环，并使用如下命令生成Monitor的密钥：

```bash
$ ceph-authtool --create-keyring ./ceph.mon.keyring --gen-key -n mon. --cap mon 'allow *'
```

创建一个 client.admin 用户 ，并将其添加到密钥中:

```bash
$ sudo ceph-authtool --create-keyring /etc/ceph/ceph.client.admin.keyring --gen-key -n client.admin --cap mon 'allow ' --cap osd 'allow ' --cap mds 'allow ' --cap mgr 'allow '
```

创建一个引导 osd 密钥环，创建一个 client.bootstrap-osd 用户并将用户添加到密钥环中：

```bash
$ ceph-authtool --create-keyring /var/lib/ceph/bootstrap-osd/ceph.keyring --gen-key -n client.bootstrap-osd --cap mon 'profile bootstrap-osd'
```

将密钥添加到 ceph.mon.keyring：

```bash
$ sudo ceph-authtool ./ceph.mon.keyring --import-keyring /etc/ceph/ceph.client.admin.keyring
$ sudo ceph-authtool ./ceph.mon.keyring --import-keyring /var/lib/ceph/bootstrap-osd/ceph.keyring
```

使用主机名、主机IP地址和FSID生成monmap，把它保存成 monmap：

```bash
$ sudo monmaptool --create --add fueltank-1 172.20.20.162 --fsid 2b19a4b1-cf7d-4eef-8e4d-eed63734d56d monmap
```

为 monitor 创建类似 /path/cluster_name-monitornode 式的目录:

```bash
$ sudo mkdir /var/lib/ceph/mon/ceph-fueltank-1
```

初始化mon：

```bash
$ sudo ceph-mon --mkfs -i fueltank-1 --monmap monmap --keyring ceph.mon.keyring
```

查看刚才创建的目录：

```bash
$ sudo ls /var/lib/ceph/mon/ceph-fueltank-1
keyring  kv_backend  store.db
```

修改权限：

```bash
$ sudo chown -R ceph:ceph /var/lib/ceph/mon/ceph-fueltank-1
```

启动 mon：

```bash
$ sudo systemctl enable ceph-mon@fueltank-1
$ sudo systemctl start ceph-mon@fueltank-1
```











