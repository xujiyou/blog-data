# Hive 架构

官方文档：https://cwiki.apache.org/confluence/display/Hive/Design

架构图：

![img](https://cwiki.apache.org/confluence/download/attachments/27362072/system_architecture.png?version=1&modificationDate=1414560669000&api=v2)

## Hive 架构

图1显示了Hive的主要组件及其与Hadoop的交互。如图所示，Hive的主要组件是：

- UI 用户界面，用于向系统提交查询和其他操作。截至2011年，该系统具有命令行界面，并且正在开发基于Web的GUI。
- Driver 接收查询的组件。该组件实现了会话句柄的概念，并提供了以JDBC / ODBC接口为模型的执行和获取API。
- Compiler 解析查询的组件，对不同的查询块和查询表达式进行语义分析，并最终借助表和从metastore查找的分区元数据来生成执行计划。
- Metastore 该组件存储仓库中各种表和分区的所有结构信息，包括列和列类型信息，读写数据所需的序列化器和反序列化器以及存储数据的相应HDFS文件。
- Execution Engine 执行由编译器创建的执行计划的组件。该计划是阶段的DAG。执行引擎管理计划的这些不同阶段之间的依赖性，并在适当的系统组件上执行这些阶段。

图1还显示了典型查询如何在系统中流动。 UI调用驱动程序的执行接口（图1中的步骤1）。驱动程序为查询创建会话句柄，并将查询发送给编译器以生成执行计划（步骤2）。编译器从元存储中获取必要的元数据（步骤3和4）。该元数据用于对查询树中的表达式进行类型检查，以及根据查询谓词修剪分区。编译器生成的计划（步骤5）是阶段的DAG，每个阶段都是映射/归约作业，元数据操作或HDFS上的操作。对于map / reduce阶段，该计划包含地图运算符树（在映射器上执行的运算符树）和reduce运算符树（对于需要reducers的运算）。执行引擎将这些阶段提交给适当的组件（步骤6、6.1、6.2和6.3）。在每个任务（映射器/化简器）中，与表或中间输出关联的解串器用于从HDFS文件读取行，并将这些行通过关联的运算符树传递。生成输出后，将通过序列化器将其写入临时HDFS文件中（如果不需要减少操作，则在映射器中发生）。临时文件用于向计划的后续地图/缩小阶段提供数据。对于DML操作，最终的临时文件将移动到表的位置。此方案用于确保不读取脏数据（文件重命名是HDFS中的原子操作）。对于查询，执行引擎直接从HDFS中读取临时文件的内容，作为驱动程序获取调用的一部分（步骤7、8和9）。



## Hive 数据模型

Hive中的数据组织为：

- Tables 这些类似于关系数据库中的表。可以对表进行过滤，投影，联接和联合。此外，表的所有数据都存储在HDFS的目录中。 Hive还支持外部表的概念，其中可以通过为表创建DDL提供适当的位置来在HDFS中预先存在的文件或目录上创建表。表中的行被组织为类似于关系数据库的类型化列。
- Partitions 每个表可以具有一个或多个确定数据存储方式的分区键，例如，具有日期分区列 ds 的表T在 <table location>/ds = <date> 目录中存储了具有特定日期数据的文件。在HDFS中。分区允许系统根据查询谓词来修剪要检查的数据，例如，对满足条件谓词T.ds='2008-09-01' 的 T 中的行感兴趣的查询，只需查看 <表位置>/ds = 2008-09-01/HDFS中的目录。
- Buckets 每个分区中的数据又可以根据表中列的哈希值划分为存储桶。每个存储段均作为文件存储在分区目录中。使用存储桶，系统可以有效地评估依赖于数据样本的查询（这些查询使用表上的SAMPLE子句）。

除了原始列类型（整数，浮点数，通用字符串，日期和布尔值）之外，Hive还支持数组和映射。此外，用户可以从任何基元，集合或其他用户定义的类型中以编程方式组合自己的类型。打字系统与SerDe（序列化/反序列化）和对象检查器接口紧密相关。用户可以通过实现自己的对象检查器来创建自己的类型，并使用这些对象检查器来创建自己的SerDes以将其数据序列化和反序列化为HDFS文件。当了解其他数据格式和更丰富的类型时，这两个接口提供了必要的钩子，以扩展Hive的功能。诸如ListObjectInspector，StructObjectInspector和MapObjectInspector之类的内置对象检查器提供了必要的原语，以可扩展的方式组成更丰富的类型。对于映射（关联数组）和数组，提供了有用的内置函数，例如大小和索引运算符。点分符号用于导航嵌套类型，例如a.b.c = 1查看类型a的字段b的字段c并将其与1进行比较。



## Metastore

Metastore提供了数据仓库的两个重要但经常被忽略的功能：数据抽象和数据发现。没有Hive中提供的数据抽象，用户必须提供有关数据格式，提取器和加载器的信息以及查询。在Hive中，此信息在表创建期间给出，并在每次引用表时重新使用。这与传统的仓储系统非常相似。第二种功能是数据发现，使用户能够发现和探索仓库中的相关数据和特定数据。可以使用此元数据构建其他工具，以公开并可能增强有关数据及其可用性的信息。 Hive通过提供与Hive查询处理系统紧密集成的元数据存储库来实现这两个功能，从而使数据和元数据同步。



#### Metadata Objects

- Database 表的名称空间。将来可以用作管理单位。数据库“default”用于没有用户提供的数据库名称的表。
- Table 表的元数据包含列，所有者，存储和SerDe信息的列表。它还可以包含用户提供的任何键和值数据。存储信息包括基础数据的位置，文件输入和输出格式以及存储信息。 SerDe元数据包括序列化程序和反序列化程序的实现类，以及实现所需的任何支持信息。可以在创建表期间提供所有这些信息。
- Partition 每个分区可以具有自己的列以及SerDe和存储信息。这有助于模式更改，而不会影响较旧的分区。



#### Metastore 架构

Metastore 是具有数据库或文件支持存储的对象存储。数据库支持的存储使用称为 DataNucleus 的对象关系映射（ORM）解决方案实现。将其存储在关系数据库中的主要动机是元数据的可查询性。使用单独的数据存储存储元数据而不使用HDFS的一些缺点是同步和可伸缩性问题。另外，由于缺乏对文件的随机更新，因此没有明确的方法在HDFS之上实现对象存储。这再加上关系存储的可查询性的优点，使我们的方法变得明智。

可以将元存储库配置为以多种方式使用：远程和嵌入式。在远程模式下，Metastore是Thrift服务。此模式对非Java客户端很有用。在嵌入式模式下，Hive客户端使用JDBC直接连接到基础元存储。此模式很有用，因为它避免了需要维护和监视的另一个系统。这两种模式可以共存。 （更新：本地Metastore是第三种可能性。有关详细信息，请参见 [Hive Metastore Administration](https://cwiki.apache.org/confluence/display/Hive/AdminManual+MetastoreAdmin)）



#### Metastore 接口

Metastore 提供了 Thrift 接口来操纵和查询 Hive 元数据。 Thrift 提供许多流行语言的绑定。第三方工具可以使用此接口将 Hive 元数据集成到其他业务元数据存储库中。



## Hive 查询语言

- Parser 将查询字符串转换为解析树表示形式。
- Semantic Analyser，语义分析器， 将解析树转换为内部查询表示形式，该表示形式仍基于块而不是运算符树。作为此步骤的一部分，将验证列名并执行*之类的扩展。在此阶段还执行类型检查和任何隐式类型转换。如果所考虑的表是分区表（这是常见情况），那么将收集该表的所有表达式，以便以后可以使用它们修剪不需要的分区。如果查询指定了采样，则也将收集该采样以供以后使用。
- Logical Plan Generator，逻辑计划生成器，将内部查询表示形式转换为由运算符树组成的逻辑计划。一些运算符是关系代数运算符，例如“ filter”，“ join”等。但是某些运算符特定于Hive，以后用于将该计划转换为一系列map-reduce作业。这样的一个运算符是reduceSink运算符，它发生在map-reduce边界处。此步骤还包括优化程序，以转换计划以提高性能-其中一些转换包括：将一系列联接转换为单个多路联接，为分组依据执行地图端部分聚合，对分组进行执行，分两个阶段进行操作，以避免单个归约器在存在分组密钥的倾斜数据的情况下成为瓶颈的情况。每个运算符都包含一个描述符，该描述符是可序列化的对象。
- Query Plan Generator，查询计划生成器，将逻辑计划转换为一系列的map-reduce任务。递归遍历运算符树，将其分解为一系列map-reduce可序列化任务，这些任务随后可提交给Hadoop分布式文件系统的map-reduce框架。 reduceSink运算符是map-reduce边界，其描述符包含归约键。 reduceSink描述符中的reduce键用作map-reduce边界中的reduce键。如果查询指定，则该计划包含所需的样本/分区。计划被序列化并写入文件。



## 优化器

优化器将执行更多计划转换。优化器是一个不断发展的组件。截至2011年，它是基于规则的，并执行以下操作：列修剪和谓词下推。但是，基础结构已经到位，并且正在进行包括其他优化（如地图侧联接）的工作。 （配置单元0.11添加了一些联接优化。）

可以将优化器增强为基于成本的（请参阅Hive和HIVE-5775中基于成本的优化）。输出表的排序性质也可以保留，以后再用于生成更好的计划。可以对少量数据样本执行查询，以猜测数据分布，这可以用于生成更好的计划。























































