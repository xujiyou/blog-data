# ceph-deploy 安装集群

ceph-deploy 安装集群的方式比较流行，不过官方决定放弃这种方式了。。。

在这里为了学习，还是学一下吧。

官方教程：https://docs.ceph.com/docs/master/install/ceph-deploy/quick-ceph-deploy/

## 安装

首先安装 ceph-deploy：

```bash
$ sudo yum install ceph-deploy
```

更新 ceph-deploy ：

```bash
$ pip install --upgrade ceph-deploy
```

添加 `/etc/yum.repos.d/ceph.repo`：

```bash
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

root 用户之间要配置免密，过程省略

安装集群：

```bash
$ sudo ceph-deploy new fueltank-1 fueltank-2 fueltank-3 # 会在当前目录生成三个配置文件
$ sudo yum -y install ceph ceph-radosgw
$ sudo ceph-deploy install --no-adjust-repos fueltank-1 fueltank-2 fueltank-3
```

完成后，每台机器上的 `ceph-crash` 服务都会启动起来：

```bash
$ systemctl status ceph-crash
```

然后：

```bash
$ sudo ceph-deploy mon create-initial
```

这步完成后，会在当前目录生成以下文件：

- `ceph.client.admin.keyring`
- `ceph.bootstrap-mgr.keyring`
- `ceph.bootstrap-osd.keyring`
- `ceph.bootstrap-mds.keyring`
- `ceph.bootstrap-rgw.keyring`
- `ceph.bootstrap-rbd.keyring`
- `ceph.bootstrap-rbd-mirror.keyring`

然后再执行：

```bash
$ sudo ceph-deploy admin fueltank-1 fueltank-2 fueltank-3
```

这会把 `ceph.client.admin.keyring` 文件拷贝到各主机的 `/etc/ceph` 目录。

创建管理守护程序：

```bash
$ sudo ceph-deploy mgr create fueltank-1
```

创建 OSD，磁盘需要自己准备，`fdisk -l` 查看磁盘，我这里创建三个 OSD:

```bash
$ sudo ceph-deploy osd create --data /dev/vdb fueltank-1
$ sudo ceph-deploy osd create --data /dev/vdb fueltank-2
$ sudo ceph-deploy osd create --data /dev/vdb fueltank-3
```

查看状态：

```bash
$ sudo ceph health
$ sudo ceph -s
```



## 卸载

```bash
$ sudo ceph-deploy purge fueltank-1 fueltank-2 fueltank-3
$ sudo ceph-deploy purgedata fueltank-1 fueltank-2 fueltank-3
$ sudo ceph-deploy forgetkeys
```

