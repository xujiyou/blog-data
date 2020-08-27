# Ceph 日志配置

官方文档：https://ceph.readthedocs.io/en/latest/rados/configuration/journal-ref/

这里说的日志是指 journal。

Ceph OSD使用日志的原因有两个：速度和一致性。

- 速度：该日志使Ceph OSD守护进程能够快速提交少量写入。Ceph 顺序地向日志写入较小的随机 I/O，这通过允许备用文件系统有更多时间合并写入，从而倾向于加快突发性工作负载。但是，Ceph OSD守护程序的日志会导致性能突飞猛进，因为短时间的高速写操作会导致文件系统追上日志而导致一段时间后没有任何写进度。
- 一致性：Ceph OSD守护程序需要保证原子复合操作的文件系统接口。Ceph OSD守护程序将操作描述写入日志，并将该操作应用于文件系统。这样可以对对象进行原子更新（例如，放置组元数据）。`filestore max sync interval` 和 `filestore min sync interval` Ceph OSD守护程序停止写操作并将日志与文件系统同步，从而使Ceph OSD守护程序可以减少日志中的操作并重新使用空间。失败时，Ceph OSD守护进程将从上一次同步操作开始重播日志。

Ceph OSD守护程序支持以下日志设置：

`journal dio`

- Description

  Enables direct i/o to the journal. Requires `journal block align` set to `true`.

- Type：Boolean

- Required：Yes when using `aio`.

- Default：`true`

`journal aio`

*Changed in version 0.61:* Cuttlefish

- Description

  Enables using `libaio` for asynchronous writes to the journal. Requires `journal dio` set to `true`.

- Type：Boolean

- Required：No.

- Default：Version 0.61 and later, `true`. Version 0.60 and earlier, `false`.

`journal block align`

- Description

  Block aligns write operations. Required for `dio` and `aio`.

- Type：Boolean

- Required：Yes when using `dio` and `aio`.

- Default：`true`

`journal max write bytes`

- Description

  The maximum number of bytes the journal will write at any one time.

- Type：Integer

- Required：No

- Default：`10 << 20`

`journal max write entries`

- Description

  The maximum number of entries the journal will write at any one time.

- Type：Integer

- Required：No

- Default：`100`

`journal queue max ops`

- Description

  The maximum number of operations allowed in the queue at any one time.

- Type：Integer

- Required：No

- Default：`500`

`journal queue max bytes`

- Description

  The maximum number of bytes allowed in the queue at any one time.

- Type：Integer

- Required：No

- Default：`10 << 20`

`journal align min size`

- Description

  Align data payloads greater than the specified minimum.

- Type：Integer

- Required：No

- Default：`64 << 10`

`journal zero on create`

- Description

  Causes the file store to overwrite the entire journal with `0`’s during `mkfs`.

- Type：Boolean

- Required：No

- Default：`false`

