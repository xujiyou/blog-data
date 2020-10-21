# Spark SQL



## Spark SQL 与 Hive 

在 spark-sql 中，是可以看到 Hive 的表的，因为 Spark SQL 和 Hive 都是使用的一个 MeteStore 服务！但是 Hive 的表默认是支持事务的，但是 spark-sql 不支持事务。

可以创建一个不带事务的表：

```sql
CREATE TABLE employee (id int, name string, salary int)
STORED AS ORC TBLPROPERTIES ('transactional' = 'false');
```

