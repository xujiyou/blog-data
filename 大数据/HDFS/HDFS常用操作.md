# HDFS 常用操作

拉数据

```bash
$ hdfs dfs -get /user/abc/ ./
```

查看大小

```bash
$ hdfs dfs -du -s -h /apps/hbase
504.6 G  1.5 T  /apps/hbase
```

第一列标示该目录下总文件大小

第二列标示该目录下所有文件在集群上的总存储大小和你的副本数相关，我的副本数是3 ，所以

第二列的是第一列的三倍 （第二列内容=文件大小*副本数）

上传

```bash
$ hdfs dfs -put file /user/abc/
```

移动数据：

```bash
$ hadoop fs -mv < hdfs file >  < hdfs file >
```



## 修改 HDFS 目录权限

```
su - hdfs -s /bin/bash -c "hadoop fs -chmod 755 /user/spark"
```

