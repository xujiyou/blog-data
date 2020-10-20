# Hive 分区

为了提高性能，我们可以在Hive中实现数据分区。在这种情况下，我们将根据日期字段创建一个带有分区列的表。仅查询所需的分区

创建外部表：

```
CREATE EXTERNAL TABLE history_raw (
    user_id STRING,
    datetime TIMESTAMP,
    ip STRING,
    browser STRING,
    os STRING)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/user/admin/input';
```

查看表结构：

```
desc formatted history_raw;
```

`/tmp/history.txt` 内容如下：

```
eo90133cf9ql,2015-10-01 00:03:20,123.456.77.88,firefox,linux,20151001
eo90133cf9ql,2015-10-01 01:08:56,123.456.77.88,firefox,linux,20151001
eo90133cf9ql,2015-10-02 02:30:45,123.456.77.88,firefox,linux,20151002
sh1243ihn93n,2015-10-02 11:21:50,121.956.23.88,safari,mac,20151002
eo90133cf9ql,2015-10-10 15:02:11,133.555.23.88,firefox,android,20151010
aa9871kjn3l1,2015-10-10 18:20:43,155.215.23.88,chrome,windows,20151010
eo90133cf9ql,2015-10-11 12:18:09,123.456.23.88,firefox,android,20151011
eo90133cf9ql,2015-10-12 12:34:34,123.456.23.88,firefox,android,20151012
hh2342o2nkj4,2015-10-15 15:02:11,133.555.23.88,safari,ios,20151015
sh1243ihn93n,2015-10-15 21:21:21,121.956.23.88,safari,mac,20151015
```

传到 HDFS：

```bash
$ hdfs dfs -put /tmp/history.txt input/
```

这时在 Hive 中查询发现有数据了：

```
select * from history_raw;
+----------------------+------------------------+-----------------+----------------------+-----------------+
| history_raw.user_id  |  history_raw.datetime  | history_raw.ip  | history_raw.browser  | history_raw.os  |
+----------------------+------------------------+-----------------+----------------------+-----------------+
| eo90133cf9ql         | 2015-10-01 00:03:20.0  | 123.456.77.88   | firefox              | linux           |
| eo90133cf9ql         | 2015-10-01 01:08:56.0  | 123.456.77.88   | firefox              | linux           |
| eo90133cf9ql         | 2015-10-02 02:30:45.0  | 123.456.77.88   | firefox              | linux           |
| sh1243ihn93n         | 2015-10-02 11:21:50.0  | 121.956.23.88   | safari               | mac             |
| eo90133cf9ql         | 2015-10-10 15:02:11.0  | 133.555.23.88   | firefox              | android         |
| aa9871kjn3l1         | 2015-10-10 18:20:43.0  | 155.215.23.88   | chrome               | windows         |
| eo90133cf9ql         | 2015-10-11 12:18:09.0  | 123.456.23.88   | firefox              | android         |
| eo90133cf9ql         | 2015-10-12 12:34:34.0  | 123.456.23.88   | firefox              | android         |
| hh2342o2nkj4         | 2015-10-15 15:02:11.0  | 133.555.23.88   | safari               | ios             |
| sh1243ihn93n         | 2015-10-15 21:21:21.0  | 121.956.23.88   | safari               | mac             |
+----------------------+------------------------+-----------------+----------------------+-----------------+
```

创建一个带分区的表：

```
CREATE TABLE history (
    user_id STRING,
    datetime TIMESTAMP,
    ip STRING,
    browser STRING,
    os STRING)
PARTITIONED BY (day STRING)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ',';
```

查看 history 的表结构 ：

```
DESC history;
+--------------------------+------------+----------+
|         col_name         | data_type  | comment  |
+--------------------------+------------+----------+
| user_id                  | string     |          |
| datetime                 | timestamp  |          |
| ip                       | string     |          |
| browser                  | string     |          |
| os                       | string     |          |
| day                      | string     |          |
|                          | NULL       | NULL     |
| # Partition Information  | NULL       | NULL     |
| # col_name               | data_type  | comment  |
| day                      | string     |          |
+--------------------------+------------+----------+
```

创建分区：

```sql
ALTER TABLE history ADD PARTITION (day='20151015');
```

查看分区：

```sql
SHOW PARTITIONS history;
+---------------+
|   partition   |
+---------------+
| day=20151015  |
+---------------+
```

插入数据：

```
INSERT OVERWRITE TABLE history PARTITION (day='20151015')
SELECT * FROM history_raw
  WHERE substr(datetime, 0, 10) = '2015-10-15';
```

查看 HDFS 中的数据组织方式：

```bash
$ hdfs dfs -ls /warehouse/tablespace/managed/hive/history
drwxrwx---+  - hive hadoop  /warehouse/tablespace/managed/hive/history/day=20151015
```



## 桶

创建一个表：

```
CREATE TABLE history_buckets (
    user_id STRING,
    datetime TIMESTAMP,
    ip STRING,
    browser STRING,
    os STRING)
CLUSTERED BY (user_id) INTO 10 BUCKETS
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ',';
```

这个表会根据 user_id 分成 10 个目录！

插入数据：

```
INSERT OVERWRITE TABLE history_buckets
SELECT * FROM history_raw;
```

查看 HDFS 中的目录，发现有10个桶：

```
$ hdfs dfs -ls /warehouse/tablespace/managed/hive/history_buckets/base_0000001
-rw-rw----+  3 hive hadoop  /warehouse/tablespace/managed/hive/history_buckets/base_0000001/bucket_00000
-rw-rw----+  3 hive hadoop  /warehouse/tablespace/managed/hive/history_buckets/base_0000001/bucket_00001
-rw-rw----+  3 hive hadoop  /warehouse/tablespace/managed/hive/history_buckets/base_0000001/bucket_00002
-rw-rw----+  3 hive hadoop  /warehouse/tablespace/managed/hive/history_buckets/base_0000001/bucket_00003
-rw-rw----+  3 hive hadoop  /warehouse/tablespace/managed/hive/history_buckets/base_0000001/bucket_00004
-rw-rw----+  3 hive hadoop  /warehouse/tablespace/managed/hive/history_buckets/base_0000001/bucket_00005
-rw-rw----+  3 hive hadoop  /warehouse/tablespace/managed/hive/history_buckets/base_0000001/bucket_00006
-rw-rw----+  3 hive hadoop  /warehouse/tablespace/managed/hive/history_buckets/base_0000001/bucket_00007
-rw-rw----+  3 hive hadoop  /warehouse/tablespace/managed/hive/history_buckets/base_0000001/bucket_00008
-rw-rw----+  3 hive hadoop  /warehouse/tablespace/managed/hive/history_buckets/base_0000001/bucket_00009
```















