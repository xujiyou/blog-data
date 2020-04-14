# Ceph 对象网关配置

Ceph 对象网关，即 RGW，是 Ceph 存储集群的一个客户端。

在  [ceph-deploy安装集群.md](../ceph-deploy安装集群.md) 中，已经学习了使用 `rados` 命令来操作对象。RGW 是另一种操作对象的方式。

Ceph对象网关运行在 Civetweb 上，Civetweb 是嵌入在 `ceph-radosgw` 守护进程中的。

```bash
$ sudo systemctl status ceph-radosgw@rgw.fueltank-1
```

 Civetweb `7480`默认在端口上运行，可以直接访问：

```bash
$ curl http://fueltank-1:7480
```



## 更改端口

编辑配置文件 `ceph.conf ` ，注意这个文件是通过 `ceph-deploy new` 生成的，而不是 etc 目录下的配置文件。

加入以下代码:

```
[client.rgw.fueltank-1]
rgw_frontends="civetweb port=82"
```

其中 fueltank-1 是我的主机名，通过 `hostname -s` 获取。

将更新的配置文件推送到 Ceph 对象网关节点（和其他Ceph节点）：

```bash
$ sudo ceph-deploy --overwrite-conf config push fueltank-1 fueltank-2 fueltank-3
```

上面这条配置会将新配置推送到各主机的 /etc/ceph 目录。。。

重启 `ceph-radosgw` 服务：

```bash
$ sudo systemctl restart ceph-radosgw@rgw.fueltank-1
```

但是事实证明这并不能成功。。。。。。。。。。













