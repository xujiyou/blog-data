# Ceph 块储存快照

官方教程：https://docs.ceph.com/docs/master/rbd/rbd-snapshot/

列出块设备：

```bash
$ sudo rbd ls kubernetes
csi-vol-73b40b6d-82d2-11ea-87c6-0a580a2a0035
```

创建快照：

```bash
$ sudo rbd snap create kubernetes/csi-vol-73b40b6d-82d2-11ea-87c6-0a580a2a0035@k8s-snapname
```

查看快照列表：

```bash
$ sudo rbd snap ls kubernetes/csi-vol-73b40b6d-82d2-11ea-87c6-0a580a2a0035
```

从快照中恢复数据（谨慎操作）：

```bash
$ sudo rbd snap rollback kubernetes/csi-vol-73b40b6d-82d2-11ea-87c6-0a580a2a0035@k8s-snapname
```

删除快照：

```bash
$ sudo rbd snap rm kubernetes/csi-vol-73b40b6d-82d2-11ea-87c6-0a580a2a0035@k8s-snapname
```

注意：Ceph OSD异步删除数据，因此删除快照不会立即释放磁盘空间。

要删除所有快照：

```bash
$ sudo rbd snap purge kubernetes/csi-vol-73b40b6d-82d2-11ea-87c6-0a580a2a0035
```



## 分层

Ceph支持创建块设备快照的许多写时复制（COW）克隆的功能。

步骤：创建快照 ---> 保护快照 ---> 克隆快照

克隆访问父快照。如果用户无意中删除了父快照，则所有克隆都将中断。为防止数据丢失，**必须**在克隆快照之前保护快照。

```bash
$ sudo rbd snap protect kubernetes/csi-vol-73b40b6d-82d2-11ea-87c6-0a580a2a0035@k8s-snapname
```

要克隆快照，请指定您需要指定父池，映像和快照。以及子池和映像名称。

```bash
$ sudo rbd clone kubernetes/csi-vol-73b40b6d-82d2-11ea-87c6-0a580a2a0035@k8s-snapname kubernetes/csi-first
```

您可以将快照从一个池克隆到另一个池中的映像。例如，您可以在一个池中将只读映像和快照作为模板维护，而在另一个池中将其可写克隆维护。

列出快照子集：

```bash
$ sudo rbd children kubernetes/csi-vol-73b40b6d-82d2-11ea-87c6-0a580a2a0035@k8s-snapname
```

查看 rbd 列表：

```bash
$ sudo rbd ls kubernetes
```

如果想保住克隆，删除快照，需要先 flatten （展平）克隆：

```bash
$ sudo rbd flatten kubernetes/csi-first
```

取消快照保护：

```bash
$ sudo rbd snap unprotect kubernetes/csi-vol-73b40b6d-82d2-11ea-87c6-0a580a2a0035@k8s-snapname
```

删除快照：

```bash
$ sudo rbd snap rm kubernetes/csi-vol-73b40b6d-82d2-11ea-87c6-0a580a2a0035@k8s-snapname
```

删除展平后的克隆，和删除块设备一样：

```bash
$ sudo rbd rm kubernetes/csi-first
```

搞定。。。



