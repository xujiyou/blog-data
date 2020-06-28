---
title: MySQL双主
date: 2020-06-28 16:02:46
tags:
---

有很多方法可以实现 MySQL 的双主结构：keepalived+双主、MHA、PXC、MMM、Heartbeat+DRBD等，比较常用的是keepalived+双主，MHA和PXC。

本文使用 Keepalived+mysql 双主来实现MySQL-HA，我们必须保证两台MySQL数据库的数据完全一样，基本思路是两台MySQL互为主从关系，通过Keepalived配置虚拟IP，实现当其中的一台MySQL数据库宕机后，应用能够自动切换到另外一台MySQL数据库，保证系统的高可用。



## 配置

master1 的相关配置如下：

```properties
log-bin=master-bin
log-bin-index=master-bin.index
relay-log=relay-bin
relay-log-index=slave-relay-bin.index
binlog-format=MIXED
server-id=1
auto-increment-increment=2
auto-increment-offset=1
```

master2 的相关配置如下：

```properties
log-bin=master-bin
log-bin-index=master-bin.index
relay-log=relay-bin
relay-log-index=slave-relay-bin.index
binlog-format=MIXED
server-id=2
auto-increment-increment=2
auto-increment-offset=2
```

master1 和 master2 只有 server-id 不同和 auto-increment-offset 不同。

mysql 中有自增长字段，在做数据库的主主同步时需要设置自增长的两个相关配置：auto_increment_offset 和 auto_increment_increment。

auto-increment-increment 表示自增长字段每次递增的量，其默认值是1。它的值应设为整个结构中服务器的总数，本案例用到两台服务器，所以值设为2。

auto-increment-offset 是用来设定数据库中自动增长的起点(即初始值)，因为这两台服务器都设定了一次自动增长值2，所以它们的起点必须得不同，这样才能避免两台服务器数据同步时出现主键冲突，

配置完成后，重启 mysqld 。



## 双主

其实这里的双主，只比主从多了一步，就是互为双主，步骤跟主从的方法一致，主从的搭建见： [MySQL离线环境主从搭建.md](MySQL离线环境主从搭建.md) 

具体步骤如下：

在 master1 和 master 2 上都执行：

```mysql
mysql> CREATE USER repl_user IDENTIFIED BY 'BBDERS1@bbdops.com';
mysql> GRANT REPLICATION SLAVE ON *.* TO repl_user;
```

在 master2 上执行：

```mysql
mysql> CHANGE MASTER TO
   		     MASTER_HOST = 'drift-1',
    	     MASTER_PORT = 3306,
    	     MASTER_USER = 'repl_user',
    	     MASTER_PASSWORD = 'BBDERS1@bbdops.com',
    	     GET_MASTER_PUBLIC_KEY = 1;
mysql> START SLAVE;
```

查看状态：

```mysql
mysql> SHOW SLAVE STATUS\G
```

如果看到：

```
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
```

说明正常。。。



在 master 1 上执行：

````mysql
mysql> CHANGE MASTER TO
   		     MASTER_HOST = 'drift-2',
    	     MASTER_PORT = 3306,
    	     MASTER_USER = 'repl_user',
    	     MASTER_PASSWORD = 'BBDERS1@bbdops.com',
    	     GET_MASTER_PUBLIC_KEY = 1;
mysql> START SLAVE;
````

查看状态：

```mysql
mysql> SHOW SLAVE STATUS\G
```

如果看到：

```
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
```

说明正常。。。



## 测试

在 master2 上创建库：

```mysql
mysql> create database drift2;
```

在 master1 上查看：

```mysql
mysql> show databases;
```

在 master1 上创建库：

```mysql
mysql> create database drift1;
```

在 master2 上查看：

```
mysql> show databases;
```



## keepalived











