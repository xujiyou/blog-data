# Ceph 新版本 PG 自动扩展

在 Ceph 15.2.4 版本中，Pool 中的 PG 数量可以自动扩展，并且这个模式是默认开启的。

参考：https://docs.ceph.com/docs/master/rados/operations/placement-groups/

查看自动扩展模式是否打开：

```bash
$ sudo ceph osd pool autoscale-status
```

