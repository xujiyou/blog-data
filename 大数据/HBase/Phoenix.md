# Phoenix

Phoenix是构建在 HBase 上的一个SQL层，能让我们用标准的 JDBC APIs 而不是 HBase 客户端 APIs 来创建表，插入数据和对HBase数据进行查询。

命令行查询：

```bash
$ phoenix-sqlline-thin hdp.testing.com
0: jdbc:phoenix:thin:url=http://hdp.testing.com> !tables
```

