# MGR 介绍

官方文档：https://ceph.readthedocs.io/en/latest/mgr/

MGR，即 Ceph 管理守护进程，进程名称为 ceph-mgr，与 Monitor 一起运行，提供其他监视和与外部监视和管理系统的接口。

自从 12.x 版本起，ceph-mgr 是必须安装的，11.x 版本之前，ceph-mgr 是可选的。

默认情况下，ceph-mgr 不需要其他配置即可运行。如果没有运行 ceph-mgr 守护程序，将看到相应的运行状况警告，并且在启动mgr之前，ceph status 输出中的某些其他信息将丢失或陈旧。

使用常规的部署工具（例如ceph-ansible或cephadm）在每个mon节点上设置ceph-mgr守护程序。将mgr守护程序与mons放在同一节点上不是强制性的，但几乎总是明智的。



## 安装

手动安装。

创建一个 密钥：

```bash
$ ceph auth get-or-create mgr.$name mon 'allow profile mgr' osd 'allow *' mds 'allow *'
```

这里 $name 一般为主机名

密钥默认应储存在 `/var/lib/ceph/mgr/ceph-$name` 中。

启动 ceph-mgr：

```bash
$ ceph-mgr -i $name
```

使用 `ceph status`查看状态，会发现以下行：

```
mgr active: $name
```





## 客户端认证

Ceph-mgr 是一个新的守护进程，默认会开启 CephX 认证功能。



## 高可用

通常，您应该在每个运行ceph-mon守护程序的主机上设置ceph-mgr，以实现相同级别的可用性。

默认情况下，无论哪个先出现的ceph-mgr实例都将由监视器激活，其他实例将成为备用数据库。在ceph-mgr守护程序之间不需要仲裁。

如果活动守护程序无法将信标发送到监视器的时间超过了`mon mgr beacon grace`（默认为30秒），那么它将被备用服务器替换。

如果您想抢先进行故障转移，则可以使用 `ceph mgr fail <mgr name>` 将ceph-mgr守护进程明确标记为失败。



## 开启 module

`ceph mgr module ls` 可以查看哪些 module 开启了，哪些没有开启，通过 `ceph mgr module enable <module>` 来开启module，通过 `ceph mgr module disable <module>` 来关闭 module。

在一个 ceph-mgr 节点上开启 module，其他节点会自动开启。

在有 HTTP 相关的 Module 时，可以使用 `ceph mgr services` 命令查看 Module 的服务地址。



## 配置

`mgr module path`

- Description：Path to load modules from
- Type：String
- Default：`"<library dir>/mgr"`

`mgr data`

- Description：Path to load daemon data (such as keyring)
- Type：String
- Default：`"/var/lib/ceph/mgr/$cluster-$id"`

`mgr tick period`

- Description：How many seconds between mgr beacons to monitors, and other periodic checks.
- Type：Integer
- Default：`5`

`mon mgr beacon grace`

- Description：How long after last beacon should a mgr be considered failed

- Type：Integer

- Default

  `30`























