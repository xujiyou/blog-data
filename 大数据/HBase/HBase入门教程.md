# HBase 入门教程

HBase 的官方文档：https://hbase.apache.org/book.html

网上也有官方文档的中文版，版本还是比较新的，比如：https://www.docs4dev.com/docs/zh/apache-hbase/2.1/reference/book.html



HBase 是一种建立在 HDFS 之上的数据库，传说中的列数据库。列数据库最适合干的就是批量数据处理，和实时查询。而传统的关系型数据库则适合处理小批量的数据，比如对一行的增删改查。

HBase 在建表时，必须要指定表名和列族。下面是建表语句：

```shell
$ hbase shell
> create 'user', 'body data', 'login data'
> create 'foo', {NAME => 'bar1', VERSIONS => 2}, {NAME => 'bar2', VERSIONS => 2}
```

这里，列簇是必须要指定的，并且后续是不可以改变的。

创建一个 user 表，user 表包含 'body data' 和 'login data' 两个列簇。

创建一个 foo 表，foo 表包含 'bar1' 和 'bar2' 两个列簇，并且版本号均为 2

列出表：

```bash
> list
```

查看表结构：

```bash
> describe 'foo'
```

插入数据 ，语法为：

```
put <table>,<rowkey>,<family:column>,<value>,<timestamp>
```

实际插入：

```bash
> put 'foo', 'row1', 'bar1:oneKey', 'oneValue'
> put 'foo', 'row1', 'bar2:twoKey', 'twoValue'
```

每次只能往一个列簇中插入。

查询数据，语法：

```
get <table>,<rowkey>,[<family:column>,....]
```

实际查询：

```bash
> get 'foo', 'row1', 'bar1'
> get 'foo', 'row1', 'bar1:oneKey'
> get 'foo', 'row1', 'bar1:oneKey', 'bar2:twoKey'
```

扫描表，限制为 5 行：

```bash
> scan 'foo', {LIMIT => 5}
```

统计行数：

```bash
> count 'foo'
```

删除：

```bash
> delete 'foo', 'row1', 'bar1:oneKey'
> delete 'foo', 'row1', 'bar1'
```

删除命令并不会立即删除数据，而是把数据标记为删除，真正的物理删除会在压缩过程中删除。



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



## 删除表

```
disable 'wanxiang:test'
drop 'wanxiang:test'
```



## 权限控制

权限类型有：

- READ("R")
- WRITE("W")
- EXEC("X")
- CREATE("C")
- ADMIN("A")

赋予权限：

```
$ sudo -u hbase hbase shell
hbase(main)> grant 'user1', 'RWXCA', 'table1'
```

查看权限：

```
hbase(main)> user_permission 'table1'
```





















