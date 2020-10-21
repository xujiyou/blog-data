# Metastore

官方文档：https://cwiki.apache.org/confluence/display/Hive/AdminManual+Metastore+3.0+Administration

Hive对象（如数据库，表和函数）的定义存储在Metastore中。根据系统的配置方式，统计信息和授权记录也可以存储在此处。 Hive和其他执行引擎在运行时使用此数据来确定如何解析，授权和有效执行用户查询。

Metastore通过DataNucleus将对象定义保存到关系数据库（RDBMS），DataNucleus是基于Java JDO的对象关系映射（ORM）层。有关可以使用的受支持RDBMS的列表，请参阅下面的受支持RDBMS。

可以将Metastore配置为嵌入Apache Derby RDBMS或连接到外部RDBMS。 Metastore本身可以完全嵌入用户进程中，也可以作为服务运行以供其他进程连接。这些选项中的每一个都将在下面依次讨论。

从Hive 3.0开始，可以在不安装其余Hive的情况下运行Metastore。它作为单独的发行版提供，以允许非Hive系统轻松与其集成。 （不过，为方便起见，它仍包含在Hive发行版中。）将Metastore变成独立服务涉及更改许多配置参数名称和工具名称。所有旧的配置参数和工具仍然有效，以便最大程度地向后兼容。本文档将涵盖旧名称和新名称。随着旧功能的添加，将不会添加Hive样式名称。



## 配置

元存储从文件 metastore-site.xml 读取其配置。为了向后兼容，它还将读取在 HIVE_HOME/conf中的任何 hive-site.xml 或 hive-metastoresite.xml 文件。

在相关部分中讨论了特定的配置值，这些配置值用于运行具有各种RDBMS（嵌入式或作为服务）且没有Hive的Metastore。以下配置值适用于Metastore，无论其运行方式如何。该表仅涵盖通常定制的配置值。

| Parameter                           | Hive 2 Parameter                   | Default Value | Description                                                  |
| :---------------------------------- | :--------------------------------- | :------------ | :----------------------------------------------------------- |
| metastore.warehouse.dir             | hive.metastore.warehouse.dir       |               | 默认目录和数据库中表的默认位置的URI。                        |
| datanucleus.schema.autoCreateAll    | datanucleus.schema.autoCreateAll   | false         | 如果不存在，则在启动时自动在RDBMS中创建必要的架构。一次创建后将其设置为false。要启用自动创建，还可以设置hive.metastore.schema.verification = false。不建议在生产中使用自动创建；改为运行`schematool`。 |
| metastore.schema.verification       | hive.metastore.schema.verification | true          | Enforce metastore schema version consistency. When set to true: verify that version information stored in the RDBMS is compatible with the version of the Metastore jar. Also disable automatic schema migration. Users are required to manually migrate the schema after upgrade, which ensures proper schema migration. This setting is strongly recommended in production. When set to false: warn if the version information stored in RDBMS doesn't match the version of the Metastore jar and allow auto schema migration. |
| metastore.hmshandler.retry.attempts | hive.hmshandler.retry.attempts     | 10            | The number of times to retry a call to the meastore when there is a connection error. |
| metastore.hmshandler.retry.interval | hive.hmshandler.retry.interval     | 2 sec         | Time between retry attempts.                                 |
| metastore.log4j.file                | hive.log4j.file                    | none          | Log4j configuration file. If unset will look for `metastore-log4j2.properties` in $METASTORE_HOME/conf |
| metastore.stats.autogather          | hive.stats.autogather              | true          | Whether to automatically gather basic statistics during insert commands. |



## RDBMS

#### Embedding Derby

该metastore可以与嵌入式Apache Derby一起运行。这是默认配置。但是，它不能用于简单测试以外的用途。在此配置中，只有一个客户端可以使用Metastore，并且任何更改都不会超出客户端的使用期限（因为它使用内存中的Derby版本）。

#### External RDBMS

对于任何持久的多用户安装，应使用外部RDBMS存储Metastore对象。 Metastore通过JDBC连接到外部RDBMS。 JDBC驱动程序为RDBMS所需的所有jar都应放在命令行中传递的METASTORE_HOME / lib中或显式。需要配置以下值以将元存储连接到RDBMS。 （注意：这些配置参数在Hive 2和3之间没有改变。）

| Configuration Parameter               | Comment                                                      |
| :------------------------------------ | :----------------------------------------------------------- |
| javax.jdo.option.ConnectionURL        | Connection URL for the JDBC driver                           |
| javax.jdo.option.ConnectionDriverName | JDBC driver class                                            |
| javax.jdo.option.ConnectionUserName   | User name to connect to the RDBMS with                       |
| javax.jdo.option.ConnectionPassword   | Password to connect to the RDBMS with. The Metastore uses [Hadoop's CredentialProvider API](http://hadoop.apache.org/docs/r3.0.1/api/org/apache/hadoop/security/alias/CredentialProvider.html) so this does not have to be stored in clear text in your configuration file. |

#### Supported RDBMSs

由于Metastore使用DataNucleus与RDBMS进行通信，因此理论上DataNucleus支持的任何存储选项都可以与Metastore一起使用。但是，我们仅测试并推荐以下内容：

| RDBMS         | Minimum Version | javax.jdo.option.ConnectionURL                       | javax.jdo.option.ConnectionDriverName          |
| :------------ | :-------------- | :--------------------------------------------------- | :--------------------------------------------- |
| MS SQL Server | 2008 R2         | jdbc:sqlserver://<HOST>:<PORT>;DatabaseName=<SCHEMA> | `com.microsoft.sqlserver.jdbc.SQLServerDriver` |
| MySQL         | 5.6.17          | jdbc:mysql://<HOST>:<PORT>/<SCHEMA>                  | `com.mysql.jdbc.Driver`                        |
| MariaDB       | 5.5             | jdbc:mysql://<HOST>:<PORT>/<SCHEMA>                  | `org.mariadb.jdbc.Driver`                      |
| Oracle*       | 11g             | jdbc:oracle:thin:@//<HOST>:<PORT>/xe                 | `oracle.jdbc.OracleDriver`                     |
| Postgres      | 9.1.13          | jdbc:postgresql://<HOST>:<PORT>/<SCHEMA>             | `org.postgresql.Driver`                        |

















