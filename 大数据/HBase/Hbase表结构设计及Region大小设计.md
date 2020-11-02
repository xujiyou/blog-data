# HBase 表结构设计及 Region 大小设计

官方文档：https://hbase.apache.org/book.html#schema

官方文档：https://hbase.apache.org/book.html#regionserver_sizing_rules_of_thumb

进行ColumnFamily修改时，必须禁用表，例如：

```java
Configuration config = HBaseConfiguration.create();
Admin admin = new Admin(conf);
TableName table = TableName.valueOf("myTable");

admin.disableTable(table);

HColumnDescriptor cf1 = ...;
admin.addColumn(table, cf1);      // adding new ColumnFamily
HColumnDescriptor cf2 = ...;
admin.modifyColumn(table, cf2);    // modifying existing ColumnFamily

admin.enableTable(table);
```

当对Tables或ColumnFamilies进行更改（例如，region大小，块大小）时，这些更改将在下次进行重大压缩时生效，并且 StoreFiles 将被重写。 有关StoreFiles的更多信息，请参见[store](https://hbase.apache.org/book.html#store) 。



## 表架构的经验法则

有许多不同的数据集，具有不同的访问模式和服务级别期望。因此，这些经验法则只是一个概述。阅读完此列表后，请阅读本章的其余部分以获取更多详细信息。

- regions 的大小在10到50 GB之间。
- cell 不超过10 MB，如果使用mob，则不超过50 MB。否则，请考虑将单元数据存储在HDFS中，并将指向数据的指针存储在HBase中。
- 典型的架构每个表具有1到3个列族。 HBase表不应设计为模仿RDBMS表。
- 对于带有1或2个列族的表，大约50-100个 regions 是一个不错的数字。请记住，regions 是列族的连续段。
- 列的限定符应尽量短。它们不应该像典型的RDBMS中那样具有自我说明性和描述性。
- 如果要存储基于时间的机器数据或日志记录信息，并且行键是基于设备ID或服务ID加上时间的，则您可能会遇到一种模式，在这种模式下，较旧的数据区域在特定使用期限内永远不会进行其他写入操作。在这种情况下，最终会出现少量活动 regions 和大量较旧的 regions，而这些 regions 没有新的写入操作。对于这些情况，您可以容忍更多区域，因为资源消耗仅由活动 regions 驱动。
- 如果只有一个列族忙于写操作，则只有该列族会占用内存。分配资源时要注意写模式。



## RegionServer大小调整规则

> 就个人而言，除非您的工作负载非常繁重，否则我将把每台机器可以为HBase专门提供的最大磁盘空间放在6T左右。在这种情况下，Java堆应为32GB（20G区域，128M内存存储，其余为默认值）。

## 关于列族的数量

HBase当前不能很好地处理超过两个或三个列族的任何事物，因此请保持架构中有尽量少的列族。当前，刷新是在每个 regions 的基础上进行的，因此，如果一个列族承载着大量要进行刷新的数据，即使相邻族族携带的数据量很小，也将对其进行刷新。当存在许多列族时，冲洗交互可以产生一堆不必要的I / O（通过更改冲洗以在每个列系列的基础上解决）。此外，在表/区域级别触发的压缩也将在每个 store 中发生。

如果可以，请尝试使用一个列族。仅在通常以列为范围的数据访问的情况下才引入第二和第三列族。也就是说，您查询一个列族或另一个列族，但通常一次都不查询。

#### ColumnFamilies的行数

在单个表中存在多个 ColumnFamilies 的地方，请注意行数。如果ColumnFamilyA有100万行，而ColumnFamilyB有10亿行，则ColumnFamilyA的数据可能会分布在许多RegionServers。这使得对ColumnFamilyA进行批量扫描的效率较低。



## Rowkey 设计

HBase中的行按行键按字典顺序排序。此设计针对扫描进行了优化，使您可以将相关的行或将一起读取的行彼此靠近存储。但是，设计不当的行键是引起热点的常见原因。当大量客户端流量定向到群集的一个节点或仅几个节点时，就会发生热点。此流量可能表示读取，写入或其他操作。流量使负责托管该区域的单台计算机不堪重负，从而导致性能下降并可能导致区域不可用。这也可能对由同一区域服务器托管的其他区域产生不利影响，因为该主机无法满足请求的负载。设计数据访问模式很重要，这样才能充分，均匀地利用群集。

为防止写入时出现热点，请设计行键，以使确实确实需要位于同一区域的行存在，但从更大的角度看，数据被写入集群中的多个区域，而不是一次写入一个区域。下面介绍了一些避免热点的常用技术，以及它们的一些优点和缺点。



#### 热点

**Salting**

从这种意义上讲，加盐与加密无关，而是指将随机数据添加到行键的开头。在这种情况下，加盐是指在行键上添加一个随机分配的前缀，以使其排序不同于其他方式。可能的前缀数量与您要在其上分布数据的区域数量相对应。如果您有一些“热”行键模式在其他分布更均匀的行中反复出现，则盐析会有所帮助。考虑下面的示例，该示例表明加盐可以将写入负载分散到多个RegionServer上，并说明对读取的某些负面影响。

> 假设您具有以下行键列表，并且对表进行了拆分，以使字母表中的每个字母都有一个区域。前缀“ a”是一个区域，前缀“ b”是另一个区域。在此表中，所有以'f'开头的行都在同一区域中。本示例重点介绍具有以下键的行：
>
> ```
> foo0001
> foo0002
> foo0003
> foo0004
> ```
>
> 现在，假设您想将它们分布在四个不同的区域。您决定使用四种不同的盐：a，b，c和d。在这种情况下，这些字母前缀中的每个字母将位于不同的区域。应用盐后，将改为使用以下行键。由于您现在可以写入四个单独的区域，因此从理论上讲，写入时的吞吐量是所有写入相同区域时的吞吐量的四倍。
>
> ```
> a-foo0003
> b-foo0001
> c-foo0004
> d-foo0002
> ```
>
> 然后，如果您添加另一行，则会随机为其分配四个可能的盐值之一，并最终靠近现有行之一。
>
> ```
> a-foo0003
> b-foo0001
> c-foo0003
> c-foo0004
> d-foo0002
> ```
>
> 由于此分配是随机的，因此，如果要按字典顺序检索行，则需要做更多的工作。这样，盐化会尝试增加写入的吞吐量，但会在读取期间增加成本。

**Hashing**

可以使用单向散列代替给定行，该散列将使给定的行始终使用相同的前缀“加盐”，从而将负载分散在RegionServer上，但允许在读取过程中进行可预测性。使用确定性哈希允许客户端重建完整的行键，并使用Get操作正常检索该行。

> 鉴于上述添加示例中的情况相同，您可以改为应用单向哈希，这将导致键为foo0003的行始终且可预测地接收到前缀。然后，要检索该行，您将已经知道密钥。您还可以优化事物，例如使某些key始终位于同一区域。

**Reversing the Key**

防止热点的第三个常见技巧是反转固定宽度或数字行键，以使变化最频繁的部分（最低有效位）在第一位。这有效地使行键随机化，但牺牲了行排序属性。



#### 单调增加行键/时间序列数据

在汤姆·怀特（Tom White）的书《 Hadoop：权威指南》（O'Reilly）的HBase章节中，有一个优化说明，注意当导入过程与所有客户协调一致地敲击表中一个区域时，这种现象（因此是单个节点），然后移至下一个区域，依此类推。随着单调增加行键（即使用时间戳），这种情况将会发生。请参阅IKai Lan的漫画，这是关于为什么单调递增的行键在类似BigTable的数据存储中会出现问题：单调递增的值是不好的。可以通过将输入记录随机化为不按排序顺序来缓解单调递增键在单个区域上的堆积，但是通常最好避免使用时间戳或序列（例如1、2、3）作为行键。

如果确实需要将时间序列数据上传到HBase，则应学习OpenTSDB作为成功的示例。它的页面描述了它在HBase中使用的架构。 OpenTSDB中的密钥格式实际上是[metric_type] [event_timestamp]，乍一看与以前关于不使用时间戳作为密钥的建议相矛盾。但是，区别在于时间戳不在密钥的主导位置，而设计假设是有数十个或数百个（或更多）不同的度量标准类型。因此，即使连续输入数据流具有多种度量标准类型，Put也会分布在表中区域的各个点上。

See [schema.casestudies](https://hbase.apache.org/book.html#schema.casestudies) for some rowkey design examples.



#### 尝试最小化行和列的大小

在HBase中，值总是随其坐标一起传送；当单元格值通过系统时，将始终伴随其行，列名和时间戳记。如果行名和列名很大，尤其是与单元格值的大小相比，则可能会遇到一些有趣的情况。 Marc Limotte在HBASE-3551的尾部描述了这种情况（推荐！）。其中，由于单元值坐标很大，保存在HBase存储文件（StoreFile（HFile））上以方便随机访问的索引可能最终会占用HBase分配的RAM的大块。上面引用的注释中的Mark表示建议增大块大小，以便存储文件索引中的条目以较大的间隔发生，或者修改表模式，从而使行和列名较小。压缩还将使索引更大。请在用户邮件列表中查看有关问题storefileIndexSize的线程。

在大多数情况下，效率低下并不重要。不幸的是，这是他们这样做的情况。无论为ColumnFamilies，属性和行键选择了哪种模式，它们都可以在数据中重复数十亿次。

有关HBase内部存储数据的更多信息，请参见[keyvalue](https://hbase.apache.org/book.html#keyvalue) ，以了解为什么这很重要。

尝试使ColumnFamily名称尽可能的小，最好是一个字符（例如，“ d”表示数据/默认值）

尽管详细的属性名称（例如“ myVeryImportantAttribute”）更易于阅读，但更喜欢将较短的属性名称（例如“ via”）存储在HBase中。

请尽量缩短它们，以使它们仍然可用于所需的数据访问（例如，获取与扫描）。对于数据访问无用的短键并不比具有更好的获取/扫描属性的长键更好。设计行键时需要权衡取舍。



long 类型为8个字节。您可以在这八个字节中存储最多18,446,744,073,709,551,615个无符号数。如果将此数字存储为字符串-（假定每个字符一个字节），则需要将近3倍的字节。

对比：

```
// long
//
long l = 1234567890L;
byte[] lb = Bytes.toBytes(l);
System.out.println("long bytes length: " + lb.length);   // returns 8

String s = String.valueOf(l);
byte[] sb = Bytes.toBytes(s);
System.out.println("long as string length: " + sb.length);    // returns 10

// hash
//
MessageDigest md = MessageDigest.getInstance("MD5");
byte[] digest = md.digest(Bytes.toBytes(s));
System.out.println("md5 digest bytes length: " + digest.length);    // returns 16

String sDigest = new String(digest);
byte[] sbDigest = Bytes.toBytes(sDigest);
System.out.println("md5 digest as string length: " + sbDigest.length);    // returns 26
```

不幸的是，使用类型的二进制表示形式会使您的数据难以在代码外部读取。例如，这是在增加值时在外壳中看到的内容：

```
hbase(main):001:0> incr 't', 'r', 'f:q', 1
COUNTER VALUE = 1

hbase(main):002:0> get 't', 'r'
COLUMN                                        CELL
 f:q                                          timestamp=1369163040570, value=\x00\x00\x00\x00\x00\x00\x00\x01
1 row(s) in 0.0310 seconds
```

shell 程序会尽力打印字符串，在这种情况下，它决定只打印十六进制。区域名称中的行键也会发生同样的情况。如果您知道要存储的内容，可以这样做，但是如果可以将任意数据放在相同的单元格中，则可能也不可读。这是主要的权衡。



#### 倒序时间戳

数据库处理中的一个常见问题是快速找到值的最新版本。使用反向时间戳作为键的一部分的技术可以在此问题的特殊情况下极大地帮助您。该技术还可以在汤姆·怀特（Tom White）的《 Hadoop：权威指南》（O'Reilly）的HBase一章中找到，该技术包括将（Long.MAX_VALUE-timestamp）附加到任何密钥的末尾，例如[key] [reverse_timestamp]。

通过执行[key]扫描并获取第一条记录，可以找到表中[key]的最新值。由于HBase键是按排序顺序排列的，因此此键在[key]的任何较旧的行键之前进行排序，因此是第一个。

使用此技术代替使用“版本号”，其目的是“永久”（或很长时间）保留所有版本，并同时使用相同的“扫描”技术快速获取对任何其他版本的访问权限。



#### 行键的不变性

行键不能更改。可以在表中“更改”它们的唯一方法是删除该行然后将其重新插入。这是HBase dist列表上的一个相当普遍的问题，因此第一次（或在插入大量数据之前）正确获取行键是值得的。



#### RowKey与Region分割之间的关系

如果您预先分割了表格，那么了解行键如何在区域边界上分布至关重要。作为说明为什么如此重要的示例，请考虑使用可显示的十六进制字符作为键的开头位置的示例（例如，“ 0000000000000000”到“ ffffffffffffffff”）。通过Bytes.split运行这些键范围（这是在Admin.createTable（byte [] startKey，byte [] endKey，numRegions）中创建区域时使用的分割策略），将生成以下10个区域...

```
48 48 48 48 48 48 48 48 48 48 48 48 48 48 48 48                                // 0
54 -10 -10 -10 -10 -10 -10 -10 -10 -10 -10 -10 -10 -10 -10 -10                 // 6
61 -67 -67 -67 -67 -67 -67 -67 -67 -67 -67 -67 -67 -67 -67 -68                 // =
68 -124 -124 -124 -124 -124 -124 -124 -124 -124 -124 -124 -124 -124 -124 -126  // D
75 75 75 75 75 75 75 75 75 75 75 75 75 75 75 72                                // K
82 18 18 18 18 18 18 18 18 18 18 18 18 18 18 14                                // R
88 -40 -40 -40 -40 -40 -40 -40 -40 -40 -40 -40 -40 -40 -40 -44                 // X
95 -97 -97 -97 -97 -97 -97 -97 -97 -97 -97 -97 -97 -97 -97 -102                // _
102 102 102 102 102 102 102 102 102 102 102 102 102 102 102 102                // f
```

（请注意：引导字节在右侧列为注释。）鉴于第一个分割为'0'，最后一个分割为'f'，一切都很好，对吗？没那么快。

问题在于所有数据都将在前两个区域和最后一个区域堆积，从而造成“块状”（可能还有“热”）区域问题。要了解原因，请参考ASCII表。 “ 0”是字节48，而“ f”是字节102，但是字节值（字节58至96）之间存在巨大差异，该间隙永远不会出现在此键空间中，因为唯一的值是[0-9]和[af ]。因此，中间区域将永远不会被使用。为了使此示例键空间可以进行预拆分，需要自定义拆分定义（即，不依赖于内置拆分方法）。

第1课：预分割表通常是最佳实践，但是您需要以可在键空间中访问所有区域的方式预分割表。尽管此示例演示了十六进制键空间的问题，但任何键空间都可能发生相同的问题。了解您的数据。

第2课：虽然通常不建议这样做，但只要在键空间中可访问所有创建的区域，使用十六进制键（更常见的是可显示的数据）仍可与预分割表一起使用。

总结这个例子，下面是一个例子，说明如何为十六进制键预先创建适当的分割：

```java
public static boolean createTable(Admin admin, HTableDescriptor table, byte[][] splits)
throws IOException {
  try {
    admin.createTable( table, splits );
    return true;
  } catch (TableExistsException e) {
    logger.info("table " + table.getNameAsString() + " already exists");
    // the table already exists...
    return false;
  }
}

public static byte[][] getHexSplits(String startKey, String endKey, int numRegions) {
  byte[][] splits = new byte[numRegions-1][];
  BigInteger lowestKey = new BigInteger(startKey, 16);
  BigInteger highestKey = new BigInteger(endKey, 16);
  BigInteger range = highestKey.subtract(lowestKey);
  BigInteger regionIncrement = range.divide(BigInteger.valueOf(numRegions));
  lowestKey = lowestKey.add(regionIncrement);
  for(int i=0; i < numRegions-1;i++) {
    BigInteger key = lowestKey.add(regionIncrement.multiply(BigInteger.valueOf(i)));
    byte[] b = String.format("%016x", key).getBytes();
    splits[i] = b;
  }
  return splits;
}
```



## 版本号

通过HColumnDescriptor为每个列系列配置了要存储的最大行版本数。最大版本的默认值为1。这是一个重要的参数，因为如“数据模型”部分所述，HBase不会覆盖行值，而是按时间（和限定符）每行存储不同的值。大型压缩期间将删除多余的版本。取决于应用程序的需要，最大版本的数量可能需要增加或减少。

不建议将最大版本数设置为过高的级别（例如，数百个或更多），除非您非常喜欢那些旧值，因为这会大大增加StoreFile的大小。

与最大行版本数一样，通过HColumnDescriptor为每个列系列配置要保留的最小行版本数。最低版本的默认值为0，这表示该功能已禁用。最小行版本数参数与生存时间参数一起使用，并且可以与行版本数参数结合使用，以进行诸如“保留最后T分钟的数据，最多N个版本，至少保留M个版本”（其中M是最小行版本数的值，M <N）。仅当为列族启用生存时间时才应设置此参数，并且该参数必须小于行版本的数目。



## 支持的数据类型

HBase通过Put和Result支持“ bytes-in / bytes-out”接口，因此任何可以转换为字节数组的内容都可以存储为值。输入可以是字符串，数字，复杂对象甚至是图像，只要它们可以呈现为字节即可。

值的大小有实际限制（例如，在HBase中存储10-50MB对象可能会要求太多）；在邮件列表中搜索有关此主题的对话。 HBase中的所有行均符合数据模型，其中包括版本控制。设计时要考虑到这一点，以及ColumnFamily的块大小。

需要特别提及的一种受支持的数据类型是“计数器”（即进行原子原子递增的能力）。请参阅表中的[Increment](https://hbase.apache.org/apidocs/org/apache/hadoop/hbase/client/Table.html#increment(org.apache.hadoop.hbase.client.Increment)) 。

计数器同步在RegionServer上完成，而不在客户端上完成。



## Joins

如果您有多个表，请不要忘记将[Joins](https://hbase.apache.org/book.html#joins)纳入架构设计的可能性。



## Time To Live (TTL)

ColumnFamilies可以设置以秒为单位的TTL长度，并且一旦达到到期时间，HBase将自动删除行。这适用于一行的所有版本-甚至是当前版本。在HBase中为该行编码的TTL时间以UTC指定。

下次压缩时将删除仅包含过期行的存储文件。将hbase.store.delete.expired.storefile设置为false将禁用此功能。将最小版本数设置为非0也会禁用此功能。

HBase的最新版本还支持设置每个单元的生存时间。有关更多信息，请参见HBASE-10560。使用mutation＃setTTL将单元TTL作为属性提交给突变请求（追加，增加，放置等）。如果设置了TTL属性，它将应用于通过该操作在服务器上更新的所有单元。Cell TTL处理和ColumnFamily TTL之间有两个显着差异：

- Cell TTL以毫秒为单位而不是秒。
- Cell TTL无法将单元的有效寿命延长到ColumnFamily级别TTL设置之外。



## 保留已删除的 Cell

默认情况下，删除标记可追溯到时间的开始。因此，即使“Get”或“Scan”操作指示放置删除标记之前的时间范围，“获取”或“扫描”操作也不会看到已删除的单元格（行或列）。

ColumnFamilies可以选择保留删除的单元格。在这种情况下，只要这些操作指定的时间范围在会影响该单元格的任何删除的时间戳记之前结束，仍然可以检索已删除的单元格。即使存在删除操作，也可以进行时间点查询。

删除的单元格仍然受TTL的约束，并且删除的单元格永远不会超过“最大版本数”。新的“原始”扫描选项将返回所有已删除的行和删除标记。

使用HBase Shell更改KEEP_DELETED_CELLS的值

```
hbase> alter ‘t1′, NAME => ‘f1′, KEEP_DELETED_CELLS => true
```

使用API更改KEEP_DELETED_CELLS的值

```
HColumnDescriptor.setKeepDeletedCells(true);
```



## 二级索引和备用查询路径

本节的标题也可能是“如果我的表行键看起来像这样，但我也想像这样查询我的表，该怎么办”。 dist列表上的一个常见示例是其中行键的格式为“ user-timestamp”，但是对于特定时间范围内的用户活动有报告要求。因此，由用户进行选择很容易，因为它处于键的引导位置，而时间却不然。

解决这一问题的最佳方法没有一个答案，因为这取决于...

- 用户数
- 数据大小和数据到达率
- 灵活的报告要求（例如，完全临时的日期选择与预先配置的范围）
- 所需的查询执行速度（例如，对于某些临时报告，90秒对于某些人来说可能是合理的，而对于另一些报告则可能太长）

解决方案还受群集大小以及必须在解决方案上投入多少处理能力的影响。常见技术在下面的小节中。这是方法的综合列表，但并不详尽。

二级索引需要额外的群集空间和处理也就不足为奇了。这正是RDBMS中发生的情况，因为创建备用索引的操作需要空间和处理周期来进行更新。 RDBMS产品在这方面更先进，可以立即处理替代索引管理。但是，HBase在较大的数据量时可更好地扩展，因此这是一个功能折衷。



#### Filter Query

根据情况，使用客户端请求过滤器可能是合适的。在这种情况下，不会创建二级索引。但是，请勿尝试从应用程序（即单线程客户端）对像这样的大表进行全面扫描。



#### 定期更新二级索引

可以在另一个表中创建二级索引，该表通过MapReduce作业定期更新。该作业可以在一天之内执行，但是根据负载策略，它仍可能与主数据表不同步。



#### 双写二级索引

另一种策略是在将数据发布到集群时构建辅助索引（例如，写入数据表，写入索引表）。如果在已存在数据表之后采用这种方法，则对于具有MapReduce作业的辅助索引，将需要进行引导（请参见[secondary.indexes.periodic](https://hbase.apache.org/book.html#secondary.indexes.periodic)）。



#### 汇总表

在时间范围很广的地方（例如，长达一年的报告），并且数据量很大，因此汇总表是一种常见的方法。这些将通过MapReduce作业生成到另一个表中。



#### 协处理器二级索引

协处理器的行为类似于RDBMS触发器。这些以0.92添加。有关更多信息，请参见[coprocessors](https://hbase.apache.org/book.html#cp)





## 约束条件

HBase当前在传统（SQL）数据库中支持“约束”。约束的建议用法是针对表中的属性执行业务规则（例如，确保值在1到10的范围内）。约束也可以用于强制执行引用完整性，但是强烈建议不要这样做，因为这会大大降低启用完整性检查的表的写吞吐量。从0.94版开始，可以在Constraint上找到有关使用[Constraint](https://hbase.apache.org/devapidocs/org/apache/hadoop/hbase/constraint/Constraint.html) 的大量文档。



































