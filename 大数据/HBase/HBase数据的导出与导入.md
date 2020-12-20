# HBase 数据的导出与导入

数据导出：

```bash
$ ./hbase org.apache.hadoop.hbase.mapreduce.Export  s1:file_data file:///ssd/hbase-backup/backup
```

然后在新库中建立相应的表结构，然后导入：

```bash
$ hbase org.apache.hadoop.hbase.mapreduce.Import s1:file_data file:///data4/hbase-backup/backup
```

