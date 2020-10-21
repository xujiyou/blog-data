# HCatalog

HCatalog 提供了一个统一的元数据服务，允许不同的工具如 Pig、MapReduce 等通过 HCatalog 直接访问存储在 HDFS 上的底层文件。

HCatalog 使用了 Hive 的元数据存储，这样就使得像 MapReduce 这样的第三方应用可以直接从 Hive 的数据仓库中读写数据。同时，HCatalog 还支持用户在 MapReduce 程序中只读取需要的表分区和字段，而不需要读取整个表。也就是提供一种逻辑上的视图来读取数据，而不仅仅是从物理文件的维度。



HCatalog 还提供了一个消息通知服务，这样对于 Oozie 这样的工作流工具，在数据仓库提供新数据时，可以通知到这些工作流工具。

HCatalog 是 Apache 的顶级项目，从 Hive0.11.0 开始，HCatalog 已经合并到 Hive 中。也就是说，如果是通过 binary 安装的 Hive0.11.0 之后的版本，HCatalog 已经自动安装了，不需要再单独部署。

因为 HCatalog 使用的就是 Hive 的元数据，因此对于 Hive 用户来说，不需要使用额外的工具来访问元数据，还是继续使用 Hive 的命令行工具。

对于非 Hive 用户，HCatalog 提供了一个称为 hcat 的命令行工具。这个工具和 Hive 的命令行工具类似，两者最大的不同就是 hcat 只接受不会产生 MapReduce 任务的命令。

如果用户需要在 MapReduce 程序中使用 HCatalog，HCatalog 提供了一个 HCatInputFormat 类来供 MapReduce 用户从 Hive 的数据仓库中读取数据。该类允许用户只读取需要的表分区和字段，同时其还以一种方便的列表格式来展示记录，这样就不需要用户来进行划分了。

同样的，HCatalog 提供了一个 HCatOutputFormat 类来供 MapReduce 用户向 Hive 中指定的表和分区中写入数据。



HCatelog 支持的 DDL 如下：

- ALTER INDEX ... REBUILD
- CREATE TABLE ... AS SELECT
- ALTER TABLE ... CONCATENATE
- ALTER TABLE ARCHIVE/UNARCHIVE PARTITION
- ANALYZE TABLE ... COMPUTE STATISTICS
- IMPORT FROM ...
- EXPORT TABLE



## hcat

HCatalog 提供了一个命令行工具，用于对元数据进行操作。

创建一个文件，命名为 temp.json，内容如下：

```
CREATE TABLE history_hcat (
    user_id STRING,
    datetime TIMESTAMP,
    ip STRING,
    browser STRING,
    os STRING)
PARTITIONED BY (day STRING)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
TBLPROPERTIES ('transactional' = 'false');
```

注意其中的 `TBLPROPERTIES ('transactional' = 'false')`，在 hcat 中，是不能创建带有 ACID 事务的表的！

执行：

```bash
$ hcat -f temp.sql 
```





























