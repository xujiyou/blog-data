# PG 管理

官方文档：https://ceph.readthedocs.io/en/latest/rados/operations/placement-groups/

PG 是 Ceph 中非常重要的概念。

## 自动扩展的 PG

系统中的每个池都有一个 `pg_autoscale_mode` 属性，可以将其设置为`off`，`on` 或 `warn`。

- `off`: 禁用此池的自动缩放。管理员可以为每个池选择适当的PG号。Please refer to [Choosing the number of Placement Groups](https://ceph.readthedocs.io/en/latest/rados/operations/placement-groups/#choosing-number-of-placement-groups) for more information.
- `on`: 启用给定池的PG计数的自动调整。
- `warn`: 当应调整PG数量时发出健康警报

要为现有池设置自动缩放模式，请执行以下操作：

```
ceph osd pool set <pool-name> pg_autoscale_mode <mode>
```

例如：

```bash
$ ceph osd pool set foo pg_autoscale_mode on
```

还可以使用以下命令配置适用于将来创建的任何池的默认pg_autoscale_mode：

```bash
$ ceph config set global osd_pool_default_pg_autoscale_mode <mode>
```



#### 查看状态

您可以使用以下命令查看每个池，池的相对利用率以及对PG计数的任何建议更改：

```bash
$ ceph osd pool autoscale-status
```





















