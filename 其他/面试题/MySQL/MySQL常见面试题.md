# MySQL 常见面试题

## MySQL 的复制原理以及流程

基本原理流程，3个线程以及之间的关联；

1. 主：binlog线程——记录下所有改变了数据库数据的语句，放进master上的binlog中；
2. 从：io线程——在使用start slave 之后，负责从master上拉取 binlog 内容，放进 自己的relay log中；
3. 从：sql执行线程——执行relay log中的语句；



## **MySQL中myisam与innodb的区别，至少5点**

1. InnoDB支持事物，而MyISAM不支持事物 
2. InnoDB支持行级锁，而MyISAM支持表级锁 
3. InnoDB支持MVCC, 而MyISAM不支持 
4. InnoDB支持外键，而MyISAM不支持 
5. InnoDB不支持全文索引，而MyISAM支持。