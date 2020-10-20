# ORC 文件格式

创建一个以 ORC 为储存格式的表：

```sql
CREATE TABLE istari (
  name STRING,
  color STRING
) STORED AS ORC;
```

修改表，以便将 istari 表的新分区存储为ORC文件：

```sql
ALTER TABLE istari SET FILEFORMAT ORC;
```

从Hive 0.14开始，用户可以通过在表或分区上发出 `CONCATENATE` 命令来请求将小型ORC文件有效合并在一起。文件将在条带级别合并，而无需重新序列化。

```
ALTER TABLE istari [PARTITION partition_spec] CONCATENATE;
```

插入数据：

```sql
insert into istari values ("xujiyou", "red");
```

要获取有关ORC文件的信息，请使用 `orcfiledump` 命令。

```bash
$ hive --orcfiledump <path_to_file>
```

比如：

```bash
$ sudo -u hdfs hive --orcfiledump hdfs://ct1.test.bbdops.com:8020/warehouse/tablespace/managed/hive/istari/delta_0000001_0000001_0000/bucket_00000
```

Hive 1.1 之后，这样查看 ORC 文件：

```bash
$ hive --orcfiledump -d <path_to_file>
```

查看表结构：

```sql
desc formatted istari;
```

查看表分区：

```
show partitions istari;
```

不是分区表会报错！



## ORC

官方文档：https://cwiki.apache.org/confluence/display/Hive/LanguageManual+ORC

*Optimized Row Columnar* ([ORC](https://orc.apache.org/)) 文件格式提供了一种高效的方式来存储Hive数据。它旨在克服其他Hive文件格式的限制。当Hive读取，写入和处理数据时，使用ORC文件可以提高性能。

与RCFile格式相比，ORC文件格式具有许多优点，例如：

- 一个文件作为每个任务的输出，从而减轻了NameNode的负担
- Hive类型支持，包括日期时间，十进制和复杂类型（结构，列表，映射和联合）
- 存储在文件中的轻量级索引
  - 跳过不通过谓词过滤的行组
  - 寻求给定的行
- 基于数据类型的块模式压缩
  - 整数列的编码
  - 字符串列的字典编码
- 使用单独的RecordReaders并发读取同一文件
- 无需扫描标记即可分割文件的功能
- 限制读取或写入所需的内存量
- 使用协议缓冲区存储的元数据，允许添加和删除字段



#### 文件结构

ORC文件包含称为条纹的行数据组，以及文件页脚中的辅助信息。在文件的末尾，附言包含压缩参数和压缩后的页脚的大小。

默认的条带大小为250 MB。大条带大小可实现从HDFS进行大而有效的读取。

文件页脚包含文件中的条带列表，每个条带的行数以及每一列的数据类型。它还包含列级聚合计数，最小，最大和总和。

下图说明了ORC文件结构：

![img](https://cwiki.apache.org/confluence/download/attachments/31818911/OrcFileLayout.png?version=1&modificationDate=1366430304000&api=v2)



#### Stripe 结构

如图所示，ORC文件中的每个条带均包含索引数据，行数据和条带脚注。

条带页脚包含流位置的目录。行数据用于表扫描。

索引数据包括每列的最小值和最大值以及每列中的行位置。 （还可以包括位字段或Bloom过滤器。）行索引条目提供偏移量，这些偏移量使能够在压缩块内寻找正确的压缩块和字节。请注意，ORC索引仅用于选择条纹和行组，而不用于回答查询。

尽管条带尺寸较大，但具有相对频繁的行索引条目可以在条带内跳过行以快速读取。默认情况下，每10,000行可以跳过。

通过基于过滤谓词跳过大行集的功能，您可以在其辅助键上对表进行排序，从而大大减少了执行时间。例如，如果主分区是交易日期，则可以按状态，邮政编码和姓氏对表进行排序。然后在一种状态下查找记录将跳过所有其他状态的记录。

格式的完整规范在[ORC specification](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+ORC#LanguageManualORC-orc-spec)中给出。



## 默认格式

hive文件存储格式包括以下几类：
1、TEXTFILE
2、SEQUENCEFILE //序列文件
3、RCFILE
4、ORCFILE(0.11以后出现)

旧版本中TEXTFILE为默认格式，建表时不指定默认为这个格式，导入数据时会直接把数据文件拷贝到hdfs上不进行处理；

新版本中 ORC 为默认格式。



















