# NFSGateway

官方文档：https://hadoop.apache.org/docs/r3.3.0/hadoop-project-dist/hadoop-hdfs/HdfsNfsGateway.html

CDH 或 HDP 上都可以添加 NFS Gateway。



安装工具：

```bash
$ sudo yum install nfs-utils
```

查看状态：

```bash
$ rpcinfo -p $nfs_server_ip
```

查看可挂载的点：

```bash
$ showmount -e $nfs_server_ip
```

挂载：

```bash
$ sudo mkdir /hdfs
$ sudo mount -t nfs -o vers=3,proto=tcp,nolock,noacl,sync $nfs_server_ip:/  /hdfs
```

这样就可以通过本地文件系统来操作HDFS了。。。

取消挂载：

```bash
$ sudo umount /hdfs
```

亲测有效

