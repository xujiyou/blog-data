# Spark SQL 入门教程

首先先在 Hive 中创建一些数据，依次执行：

```
$ sudo hive
hive> CREATE TABLE IF NOT EXISTS employee ( eid int, name String, salary String, destination String) COMMENT 'Employee details' ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' LINES TERMINATED BY '\n' STORED AS TEXTFILE;
hive> show tables;
hive> insert into employee values(1,'xujiyou','BBB', 'AAA');
hive> select * from employee;
```

然后写一些 Scala 代码：

