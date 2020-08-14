# Ceph 块设备挂载

- 参考：https://docs.ceph.com/docs/master/rbd/rbd-ko/
- 参考：https://docs.ceph.com/docs/master/start/quick-rbd/



按照  [Ceph块储存入门.md](Ceph块储存入门.md)  中的步骤创建一个块设备。

映射：

```bash
$ sudo rbd device map one/bar
```

但是报错：

```
rbd: sysfs write failed
RBD image feature set mismatch. Try disabling features unsupported by the kernel with "rbd feature disable".
In some cases useful info is found in syslog - try "dmesg | tail".
rbd: map failed: (6) No such device or address
```

执行 `dmesg | tail`，查看原因：

```
[599056.927599] libceph: mon4 172.20.20.145:3300 socket closed (con state CONNECTING)
[599066.943087] libceph: mon0 172.20.20.162:3300 socket closed (con state CONNECTING)
[599516.228996] libceph: mon3 172.20.20.179:6789 feature set mismatch, my 106b84a842a42 < server's 40106b84a842a42, missing 400000000000000
[599516.248956] libceph: mon3 172.20.20.179:6789 missing required protocol features
[599526.941551] libceph: mon4 172.20.20.145:3300 socket closed (con state CONNECTING)
[599536.925665] libceph: mon2 172.20.20.179:3300 socket closed (con state CONNECTING)
[599546.942808] libceph: mon5 172.20.20.145:6789 feature set mismatch, my 106b84a842a42 < server's 40106b84a842a42, missing 400000000000000
[599546.955721] libceph: mon5 172.20.20.145:6789 missing required protocol features
[599556.925647] libceph: mon2 172.20.20.179:3300 socket closed (con state CONNECTING)
[599566.941521] libceph: mon4 172.20.20.145:3300 socket closed (con state CONNECTING)
```

解决方法，禁用掉一些 feature。查看有哪些 feature：

```bash
$ sudo rbd info one/bar
```

返回：

```bash
rbd image 'bar':
        size 10 GiB in 2560 objects
        order 22 (4 MiB objects)
        snapshot_count: 0
        id: 169b42d588f5
        block_name_prefix: rbd_data.169b42d588f5
        format: 2
        features: layering, exclusive-lock, object-map, fast-diff, deep-flatten
        op_features: 
        flags: 
        create_timestamp: Mon Apr 20 15:48:08 2020
        access_timestamp: Mon Apr 20 16:14:29 2020
        modify_timestamp: Mon Apr 20 15:48:08 2020
```

禁掉：

```bash
$ sudo rbd feature disable one/bar exclusive-lock object-map fast-diff deep-flatten
```

再次映射，就成功了：

```bash
$ sudo rbd device map one/bar
/dev/rbd0
```

查看映射到本地的块设备：

```bash
$ sudo fdisk -l
```

使用 rbd 查看已映射的设备：

```bash
$ rbd device list
```



## 修改配置

上面为每一个块设备都禁用特性，显然不科学，这里需要修改配置。

集群使用的是 ceph-deploy 部署的，修改部署的时候生成的 ceph.conf，添加如下语句

```
rbd_default_features = 1
```

然后运行命令：

```bash
$ sudo ceph-deploy --overwrite-conf admin fueltank-1 fueltank-2 fueltank-3
```

这句命令会将配置发布到每一个节点，并使之生效。



## 挂载到文件系统

执行命令，格式化磁盘：

```bash
$ sudo mkfs.ext4 -m0 /dev/rbd0
```

等一会完成之后：

```bash
$ sudo mkdir /mnt/rbd0
$ sudo mount /dev/rbd0 /mnt/rbd0
```

关于开机自动挂载到文件系统，关机自动卸载，可以看：https://docs.ceph.com/docs/master/man/8/rbdmap/

在 `/etc/ceph/rbdmap` 文件中，添加以下内容：

```
one/bar    id=admin,keyring=/etc/ceph/ceph.client.admin.keyring
```

在/etc/fstab中，加入一下内容：

```
/dev/rbd0 /mnt/rbd0 ext4 noauto 0 0
```

然后让 radmap 开机自启：

```bash
$ sudo systemctl enable rbdmap.service
```

重启测试（慎重）：

```bash
$ sudo reboot
```

亲测可行！！！！！！就是启动慢点而已。。。。。。



## 卸载

取消挂载

```bash
$ umount /dev/rbd0
```

取消映射：

```bash
$ rbd device unmap one/bar
```

删除块设备：

```bash
$ rbd rm one/bar
```

删除 pool：

```bash
$ sudo ceph osd pool delete one one --yes-i-really-really-mean-it
```



## 踩坑

有一次，我没有 unmap，也没删块设备，但是我删了 pool，回过头来发现不能 unmap 了，也不能删块设备了，这个块设备空间也不大，但是强迫症，必须要删掉，后来是先取消了挂载，然后重启了一下机器（慎重），在 `lsblk` 里那个块设备就自动不见了，使用命令 `rbd showmapped` 也看不到那块块设备了。





