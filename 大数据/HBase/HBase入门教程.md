# HBase 入门教程

---

HBase 是一种建立在 HDFS 之上的数据库，传说中的列数据库。列数据库最适合干的就是批量数据处理，和实时查询。而传统的关系型数据库则适合处理小批量的数据，比如对一行的增删改查。

HBase 在建表时，必须要指定表名和列族。下面是建表语句：

```shell
$ hbase shell
> create 'user', 'body data', 'login data'
```

创建一个 user 表，user 表包含 'body data' 和 'login data' 两个列族

用 Java 创建表的代码如下：

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.util.Bytes;

import java.io.IOException;

public class HBaseDemo {

    public static void main(String[] args) throws IOException {
        Configuration configuration = HBaseConfiguration.create();
        configuration.set("hbase.zookeeper.quorum", "fueltank-2:2181");
        Connection connection = ConnectionFactory.createConnection(configuration);

        Admin admin = connection.getAdmin();
        TableDescriptorBuilder tableDescriptorBuilder = TableDescriptorBuilder.newBuilder(TableName.valueOf("user"));
        ColumnFamilyDescriptorBuilder bodyData = ColumnFamilyDescriptorBuilder.newBuilder(Bytes.toBytes("body data"));
        tableDescriptorBuilder.setColumnFamily(bodyData.build());
        ColumnFamilyDescriptorBuilder loginData = ColumnFamilyDescriptorBuilder.newBuilder(Bytes.toBytes("login data"));
        tableDescriptorBuilder.setColumnFamily(loginData.build());
        admin.createTable(tableDescriptorBuilder.build());
        System.out.println("创建表成功!");
    }
}
```

下面是 HBase 客户端的 Maven 依赖，可查询 Maven 库来寻找最新版本：

```xml
<dependency>
  <groupId>org.apache.hbase</groupId>
  <artifactId>hbase-client</artifactId>
  <version>2.1.0</version>
</dependency>
```

