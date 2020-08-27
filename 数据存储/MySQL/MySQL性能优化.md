---
title: MySQL性能优化
date: 2020-07-16 10:53:28
tags:
---

查看索引：

```mysql
mysql> show index from dynamic_person;
```

线上说，mysql  count 查询太慢，

找一个比较简单的带索引的字段：

```mysql
mysql> select count(type) from dynamic_person;
```

count 带索引的字段会比较快。这次花费了 134 秒。

服务器内存是够用的，想加大 mysql 的内存。

```mysql
mysql> show variables like 'innodb_buffer_pool_size';
+-------------------------+-----------+
| Variable_name           | Value     |
+-------------------------+-----------+
| innodb_buffer_pool_size | 134217728 |
+-------------------------+-----------+
1 row in set (0.00 sec)
```

134217728 = 128 * 1024 * 1024

128 MB，比较小，服务器有 256 GB，

我想设置为 128 GB。即 137438951472 

```mysql
mysql> set global innodb_buffer_pool_size=137438951472;
```

mysql 是使用 docker 部署的，查看内存：

```
$ docker stats
CONTAINER   CPU %   MEM USAGE / LIMIT       MEM %    NET I/O             BLOCK I/O           PIDS
54542b63f   16.09%  19.2 GiB / 251.7 GiB    7.63%    1.99 GB / 808 MB    0 B / 2.43 TB       69
```

设置后，`MEM USAGE` 会缓慢增长。

再次查询，发现性能提高了，再次查询，用了 13 秒，速度提高了 10 倍。





## 其他优化

```
max_connections = 1024
slow_query_log = ON
long_query_time = 5
max_allowed_packet = 104857600
```



