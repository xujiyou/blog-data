# cephadm 安装集群

这里先采用最简单的 cephadm 安装，官方教程：https://docs.ceph.com/docs/master/cephadm/install/

好像要用到 docker，提前安装。

---

## 安装

获取 cephadm 二进制文件，并放入 PATH 中：

```bash
$ curl --silent --remote-name --location https://github.com/ceph/ceph/raw/octopus/src/cephadm/cephadm
$ chmod +x cephadm
$ sudo mv cephadm /usr/bin/
```

准备数据目录：

```bash
$ sudo mkdir /mnt/vde/ceph
$ sudo ln -s /mnt/vde/ceph /var/lib/ceph
```

安装集群：

```bash
$ mkdir -p /etc/ceph
$ sudo cephadm add-repo --release octopus
$ sudo cephadm bootstrap --mon-ip 172.20.20.162 --allow-fqdn-hostname --allow-overwrite
```

安装完成后，会提示以下信息：

```
INFO:cephadm:Ceph Dashboard is now available at:

             URL: https://fueltank-1.cloud.bbdops.com:8443/
            User: admin
        Password: nyd04nqgc1

INFO:cephadm:You can access the Ceph CLI with:

        sudo /sbin/cephadm shell --fsid a7562d6e-7d2a-11ea-ba3d-fa163e968d14 -c /etc/ceph/ceph.conf -k /etc/ceph/ceph.client.admin.keyring

INFO:cephadm:Please consider enabling telemetry to help improve Ceph:

        ceph telemetry on

For more information see:

        https://docs.ceph.com/docs/master/mgr/telemetry/

INFO:cephadm:Bootstrap complete.
```

登录 Dashboard ，会要求改密码，我这里为了测试，改成了 123456789

如果想运行 `ceph`、`rbd` 等命令：

```bash
$ sudo cephadm shell
```

如果想在本机使用命令，可以通过以下方式安装：

```bash
$ sudo cephadm install ceph-common
```

几个 ceph 命令：

```bash
$ ceph -v
$ ceph status
```



---



## 添加主机

查看主机列表：

```bash
$ sudo ceph orch host ls
```

将公钥添加到其他主机的信任列表中。

```bash
$ ssh-copy-id -f -i /etc/ceph/ceph.pub root@*<new-host>*
```

也可以直接将 `/etc/ceph/ceph.pub` 中的内容添加到其他主机的 `/root/ssh/authorized_keys` 文件底部。

在其他主机上也要提前准备数据目录，因为一般系统的跟目录容量都很小。

添加主机：

```bash
$ sudo ceph orch host add fueltank-2.cloud.bbdops.com
$ sudo ceph orch host add fueltank-3.cloud.bbdops.com
```

再次查看主机列表：

```bash
$ sudo ceph orch host ls
```



---



## 创建 OSD

查看磁盘：

```bash
$ sudo ceph orch device ls
```



使用所有可用并且未使用的磁盘：

```bash
$ sudo ceph orch apply osd --all-available-devices
```

指定一个主机上的磁盘来创建：

```bash
$ sudo ceph orch daemon add osd fueltank-1.cloud.bbdops.com:/dev/vdf
```

这个磁盘必须保证未被挂载为文件系统。

查看 OSD 状态：

```bash
$ sudo ceph osd status
$ sudo ceph osd tree
```

这里要挂三个 OSD ，集群才能变健康！！！



---



## 部署 RGW

Ceph RGW(即RADOS Gateway)是Ceph对象存储网关服务，是基于LIBRADOS接口封装实现的FastCGI服务，对外提供存储和管理对象数据的Restful API。 对象存储适用于图片、视频等各类文件的上传下载，可以设置相应的访问权限。目前Ceph RGW兼容常见的对象存储API，例如兼容绝大部分Amazon S3 API，兼容OpenStack Swift API。

```bash
$ sudo radosgw-admin realm create --rgw-realm=one --default
$ sudo radosgw-admin zonegroup create --rgw-zonegroup=one-zone  --master --default
$ sudo radosgw-admin zone create --rgw-zonegroup=one-zone --rgw-zone=one-zone-name --master --default
$ sudo ceph orch apply rgw one one-zone-name 1 fueltank-1.cloud.bbdops.com
```

#### 开通 RGW Dashboard

部署了 RGW 之后，在 UI 上还不能显示，需要配置

教程：https://docs.ceph.com/docs/octopus/mgr/dashboard/#enabling-the-object-gateway-management-frontend

过程如下：

```bash
$ sudo radosgw-admin user create --uid=xujiyou_id --display-name=xujiyou --system
$ sudo radosgw-admin user info --uid=xujiyou_id
# 会展示出用户信息，其中就包括 access-key 和 secret-key
$ sudo ceph dashboard set-rgw-api-user-id xujiyou_id
$ sudo ceph dashboard set-rgw-api-access-key JYHPNEOTIQ4X1H1KXP3P
$ sudo ceph dashboard set-rgw-api-secret-key AuY69uhXTN6UMKb5WkgVf4fsSlsu2bIBuuaCnXwg
$ sudo ceph mgr module disable dashboard
$ sudo ceph mgr module enable dashboard
```



查看用户：

```bash
$ sudo radosgw-admin user list
```



---



## 卸载

```bash
$ sudo cephadm ls
$ sudo cephadm rm-cluster --fsid=6b93db78-7cb2-11ea-93bb-fa163e968d14 --force
```

但是卸载后，创建的逻辑卷不会自动删除，老恶心了。`fdisk -l` 中那个逻辑卷怎么都删不掉。。。

这里需要先重启！！！然后执行以下操作：

```bash
$ sudo lvdisplay
$ sudo lvremove /dev/ceph-e0242907-2693-4d51-a454-18aa38145020/osd-block-245236cd-4902-4973-b306-baa61bb16d1b
```

