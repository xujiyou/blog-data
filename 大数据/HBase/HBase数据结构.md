# HBase 数据结构

在HBase中，数据存储在具有行和列的表中。这是与关系数据库（RDBMS）的术语重叠，但这并不是一个有用的类比。相反，将HBase表视为多维映射可能会有所帮助。

可以将 HBase 中的数据结构想象成一个巨大的 HashMap。



#### 数据结构术语

**Table**

一个表由很多数据行组成。

**Row**

HBase中的一行由行键和一列或多列与它们相关联的值组成。行在存储时按行键字母顺序排序。因此，行键的设计非常重要。目标是以相关行彼此靠近的方式存储数据。常见的行键模式是网站域。如果行键是域，则可能应该将它们反向存储（org.apache.www，org.apache.mail，org.apache.jira）。这样，所有Apache域在表中彼此靠近，而不是根据子域的第一个字母散布开。

**Column**

HBase中的列由列族和列限定符组成，它们由:（冒号）字符分隔。

**Column Family**

出于性能考虑，列族实际上将一组列及其值并置在一起。每个列族都有一组存储属性，例如是否应将其值缓存在内存中，如何压缩其数据或对其行键进行编码等。表中的每一行都具有相同的列族，尽管给定的行可能不会在给定的列族中存储任何内容。

**Column Qualifier**

将列限定符添加到列族，以提供给定数据段的索引。给定列族内容，列限定符可能是`content:html`，另一个可能是`content:pdf` 。尽管列族在创建表时是固定的，但列限定符是可变的，并且行之间的差异可能很大。

**Cell**

单元格是行，列族和列限定符的组合，并包含一个值和一个时间戳，代表该值的版本。

**Timestamp**

时间戳记与每个值一起写入，并且是值的给定版本的标识符。默认情况下，时间戳表示写入数据时在RegionServer上的时间，但是在将数据放入单元格时，可以指定其他时间戳值。



## 概念视图

有一个名为 webtable 的表，其中包含两行（com.cnn.www 和 com.example.www）和三个列族，分别名为 content ，anchor 和 people 。在此示例中，对于第一行（com.cnn.www），anchor 包含两列（ anchor:cssnsi.com，anchor:my.look.ca），content包含一列（contents:html）。本示例包含具有行键com.cnn.www的行的5个版本，以及具有行键com.example.www的行的一个版本。 content:html列限定符包含给定网站的整个HTML。anchor 列族的限定词每个都包含链接到该行表示的站点的外部站点，以及该链接的锚定中使用的文本。people 列族代表与该站点关联的人员。

> *Column Names*
>
> 按照惯例，列名称由其列族前缀和限定符组成。例如，contents:html 列由列族 content 和 html 限定符组成。冒号分隔列族和列族限定符。

webtable 表：

| Row Key           | Time Stamp | ColumnFamily `contents`   | ColumnFamily `anchor`         | ColumnFamily `people`      |
| :---------------- | :--------- | :------------------------ | :---------------------------- | :------------------------- |
| "com.cnn.www"     | t9         |                           | anchor:cnnsi.com = "CNN"      |                            |
| "com.cnn.www"     | t8         |                           | anchor:my.look.ca = "CNN.com" |                            |
| "com.cnn.www"     | t6         | contents:html = "<html>…" |                               |                            |
| "com.cnn.www"     | t5         | contents:html = "<html>…" |                               |                            |
| "com.cnn.www"     | t3         | contents:html = "<html>…" |                               |                            |
| "com.example.www" | t5         | contents:html = "<html>…" |                               | people:author = "John Doe" |

该表中看起来为空的单元格在HBase中不占用空间，或者实际上不存在。这就是使HBase“稀疏”的原因。表格视图不是查看HBase中数据的唯一可能方法，甚至不是最准确的方法。以下代表与多维 Map 相同的信息。这仅是出于说明目的的模型，可能并非严格准确。

```
{
  "com.cnn.www": {
    contents: {
      t6: contents:html: "<html>..."
      t5: contents:html: "<html>..."
      t3: contents:html: "<html>..."
    }
    anchor: {
      t9: anchor:cnnsi.com = "CNN"
      t8: anchor:my.look.ca = "CNN.com"
    }
    people: {}
  }
  "com.example.www": {
    contents: {
      t5: contents:html: "<html>..."
    }
    anchor: {}
    people: {
      t5: people:author: "John Doe"
    }
  }
}
```



## 物理视图

尽管从概念上讲，表可以看作是行的稀疏集合，但它们实际上是按列族存储的。可以随时将新的列限定符（column_family:column_qualifier）添加到现有的列族。

列族不可以改或添加，列限定符可以随意添加！

anchor 列族：

| Row Key       | Time Stamp | Column Family `anchor`        |
| :------------ | :--------- | :---------------------------- |
| "com.cnn.www" | t9         | anchor:cnnsi.com = "CNN"      |
| "com.cnn.www" | t8         | anchor:my.look.ca = "CNN.com" |

contents 列族：

| Row Key       | Time Stamp | Column Family `anchor`    |
| :------------ | :--------- | :------------------------ |
| "com.cnn.www" | t6         | contents:html = "<html>…" |
| "com.cnn.www" | t5         | contents:html = "<html>…" |
| "com.cnn.www" | t3         | contents:html = "<html>…" |

概念视图中显示的空单元格根本不存储。因此，在时间戳记t8处对content:html列的值的请求将不返回任何值。同样，在时间戳t9处请求anchor:my.look.ca 值的请求将不返回任何值。但是，如果未提供时间戳，则将返回特定列的最新值。给定多个版本，最新的也是找到的第一个版本，因为时间戳以降序存储。因此，如果未指定时间戳，则对com.cnn.www行中所有列的值的请求为：来自时间戳t6的content:html的值，来自时间戳t9的anchor:cnnsi.com的值，时间戳记t8中的anchor:my.look.ca。

有关Apache HBase如何存储数据的内部信息的更多信息，请参见[regions.arch](https://hbase.apache.org/book.html#regions.arch).



## Namespace

命名空间是表的逻辑分组，类似于关系数据库系统中的数据库。这种抽象为即将到来的多租户相关功能奠定了基础：

- 配额管理：限制名称空间可以消耗的资源（即区域，表）数量。
- 命名空间安全管理：为租户提供另一级别的安全管理。
- Region server 组：可以将名称空间/表固定到RegionServers的子集上，从而确保隔离的粗略水平。

可以创建，删除或更改名称空间。命名空间是在表创建期间通过指定以下格式的标准表名来确定的：

```
<table namespace>:<table qualifier>
```

创建一个命名空间

```
hbase> create_namespace 'my_ns'
```

在命名空间中创建表：

```
hbase> create 'my_ns:my_table', 'fam'
```

删除命名空间：

```
hbase> drop_namespace 'my_ns'
```

alter namespace:

```
hbase> alter_namespace 'my_ns', {METHOD => 'set', 'PROPERTY_NAME' => 'PROPERTY_VALUE'}
```

#### 预定义的名称空间

有两个预定义的特殊名称空间：

hbase： 系统名称空间，用于包含HBase内部表

default：没有明确指定名称空间的表将自动属于该名称空间

例子：

```
#namespace=foo and table qualifier=bar
hbase> create 'foo:bar', 'fam'

#namespace=default and table qualifier=bar
hbase> create 'bar', 'fam'
```



## Table

表是在架构定义时预先声明的。



## Row

行键按字典顺序进行排序，其中最低顺序出现在表中。空字节数组用于表示表名称空间的开始和结束。



## Column Family

HBase 中的列被分组为列族。列族的所有列成员都具有相同的前缀。例如，列 courses:history 和 courses:math 都是 courses 列族的成员。冒号(:)分隔列族和列族限定符。列族前缀必须由可打印字符组成。限定尾部（列族限定符）可以由任意字节组成。列族必须在架构定义时预先声明，而不必在架构时定义列，但可以在表启动并运行时即时对其进行构想。

实际上，所有列族成员都一起存储在文件系统上。由于调整和存储规范是在列族级别上完成的，因此建议所有列族成员具有相同的常规访问模式和大小特征。



## Cells

{row，column，version}元组精确地指定了HBase中的一个单元。单元格内容是字节。





## Versions

可能会有无数个单元格，其中行和列相同，但单元格地址仅在其版本维度上有所不同。

当行键和列键表示为字节时，使用长整数指定版本。通常，此long包含一些时间实例，例如java.util.Date.getTime（）或System.currentTimeMillis（）返回的那些实例。

HBase版本维以降序存储，因此从存储文件中读取时，将首先找到最新值。

在HBase中，关于单元版本的语义存在很多混乱。特别是：

- 如果对一个单元的多次写入具有相同版本，则只有最后一次写入是可获取的。
- 可以按非递增版本顺序写入单元格。

在撰写本文时，HBase中不再存在本文中提到的在现有时间戳上覆盖值的限制。这部分基本上是Bruno Dumon撰写的这篇文章的提要。

#### 指定要存储的版本

给定列存储的最大版本数是列架构的一部分，并在创建表时指定，或通过 HColumnDescriptor.DEFAULT_VERSIONS 通过 alter 命令指定。在HBase 0.96之前，保留的默认版本数是3，但在0.96和更高版本中已更改为1。

修改列族的最大版本数：

```
hbase> alter ‘t1′, NAME => ‘f1′, VERSIONS => 5
```

修改列族的最小版本数：

可以指定每个列系列要存储的最小版本数。默认情况下，它设置为0，这意味着该功能被禁用。

```
hbase> alter ‘t1′, NAME => ‘f1′, MIN_VERSIONS => 2
```

从HBase 0.98.2开始，可以通过在hbase-site.xml中设置hbase.column.max.version，为所有新创建的列保留的最大版本数指定全局默认值。



## Sort Order

所有数据模型操作HBase都按排序顺序返回数据。首先是按行，然后是ColumnFamily，然后是列限定符，最后是时间戳（按相反顺序排序，因此首先返回最新记录）。



## Column Metadata

在 ColumnFamily 的内部 KeyValue 实例之外没有任何列元数据存储。因此，HBase不仅可以支持每行大量列，而且还可以支持行之间的异构列集，但是您有责任跟踪列名。

获取 ColumnFamily 存在的一组完整列的唯一方法是处理所有行。有关 HBase 如何在内部存储数据的更多信息，请参见[keyvalue](https://hbase.apache.org/book.html#keyvalue).



## Joins

HBase是否支持联接是 dist-list 上的一个常见问题，并且有一个简单的答案：不支持。如本章所述，HBase中的读取数据模型操作是Get和Scan。

但是，这并不意味着您的应用程序不支持等效的联接功能，而是您必须自己做。两种主要策略是在写入HBase 时对数据进行非规范化，或者在应用程序或MapReduce代码中具有查找表并在HBase表之间进行联接（正如RDBMS所演示的那样，有几种策略可用于此操作，具体取决于表，例如，嵌套循环与哈希联接）。那么哪种方法最好呢？这取决于您要尝试执行的操作，因此，没有一个适用于每个用例的答案。



## ACID

See [ACID Semantics](https://hbase.apache.org/acid-semantics.html). Lars Hofhansl has also written a note on [ACID in HBase](http://hadoop-hbase.blogspot.com/2012/03/acid-in-hbase.html).

















