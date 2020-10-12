# HDFS 快照

官方文档：https://hadoop.apache.org/docs/r3.3.0/hadoop-project-dist/hadoop-hdfs/HdfsSnapshots.html

HDFS快照是文件系统的只读时间点副本。可以在文件系统或整个文件系统的子树上拍摄快照。快照的一些常见用例是数据备份，防止用户错误和灾难恢复。

HDFS快照的实施非常有效：

- 快照创建是瞬时的：不包括inode查找时间，成本为O（1）。
- 仅在相对于快照进行修改时才使用附加内存：内存使用量为O（M），其中M是已修改文件/目录的数量。
- 不会复制datanode中的块：快照文件记录了块列表和文件大小。没有数据复制。
- 快照不会对HDFS的常规操作产生不利影响：修改记录以相反的时间顺序进行，因此可以直接访问当前数据。通过从当前数据中减去修改来计算快照数据。



## 快照目录

将目录设置为快照表后，即可在任何目录上拍摄快照。快照表目录能够容纳65,536个同时快照。快照表目录的数量没有限制。管理员可以将任何目录设置为快照表。如果快照表目录中有快照，则在删除所有快照之前，不能删除或重命名该目录。

目前不允许嵌套的快照表目录。换句话说，如果目录的祖先/后代之一是快照表目录，则无法将其设置为快照表。



## 快照路径

对于快照表目录，路径组件“ .snapshot”用于访问其快照。假设 /foo 是快照表目录，/foo/bar是 /foo 中的文件/目录，并且/foo 具有快照s0。然后，路径 /foo/.snapshot/s0/bar 引用 /foo/bar 的快照副本。常用的API和CLI可以使用“ .snapshot”路径。以下是一些示例。

列出snapshottable目录下的所有快照：

```bash
$ hdfs dfs -ls /foo/.snapshot
```

列出快照s0中的文件：

```bash
$ hdfs dfs -ls /foo/.snapshot/s0
```

从快照s0复制文件：

```bash
$ hdfs dfs -cp -ptopax /foo/.snapshot/s0/bar /tmp
```

请注意，此示例使用preserve选项保留时间戳，所有权，权限，ACL和XAttrs。



## 升级到带有快照的HDFS版本

HDFS快照功能引入了用于与快照进行交互的新保留路径名：.snapshot。从不支持快照的较旧版本的HDFS升级时，需要首先重命名或删除现有的名为.snapshot的路径，以避免与保留的路径发生冲突



## 快照操作

本节中描述的操作需要超级用户特权。

#### Allow Snapshots

允许创建目录快照。如果操作成功完成，该目录将成为快照表。

```bash
$ hdfs dfsadmin -allowSnapshot <path>
```



#### Disallow Snapshots

不允许创建目录的快照。必须先删除目录的所有快照，然后再禁止快照。

```bash
$ hdfs dfsadmin -disallowSnapshot <path>
```



#### Create Snapshots

创建快照表目录的快照。此操作需要快照表目录的所有者特权。

```bash
$ hdfs dfs -createSnapshot <path> [<snapshotName>]
```



#### Delete Snapshots

从快照表目录中删除快照。此操作需要快照表目录的所有者特权。

```bash
$ hdfs dfs -deleteSnapshot <path> <snapshotName>
```



#### Rename Snapshots

```bash
$ hdfs dfs -renameSnapshot <path> <oldName> <newName>
```



#### Get Snapshottable Directory Listing

获取当前用户有权拍摄快照的所有快照表目录。

```bash
$ hdfs lsSnapshottableDir
```



#### Get Snapshots Difference Report

```bash
$ hdfs snapshotDiff <path> <fromSnapshot> <toSnapshot>
```





















