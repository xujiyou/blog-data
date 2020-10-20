# MySQL 复制

参考书籍：《高可用MySQL》

单一的主从复制是比较简单的。

## 主从

分为三个简单的步骤：

1. 配置一个服务器为 master。
2. 配置一个服务器为 slave。
3. 将 slave 连接到 master。

关于单价搭建MySQL： [MySQL最新版本安装.md](../MySQL最新版本安装.md) 



## 配置 Master

要将服务器配置为 master，要确保服务器有 binlog 和 唯一的服务器ID。

在 my.cnf 中加入以下配置：

```
log-bin=master-bin
log-bin-index=master-bin.index
server-id=1
```

这个最好为 log-bin 设置一个固定的文件名，因为默认的 log-bin 带有主机名，在主机名改变后，会发生混乱。

Salve启动一个客户端连接到 Master，请求 Mater 经所有的变更发给他。Slave 连接 Master 时要求 Master 上有一个特殊复制权限的用户。

在 Master 上创建一个用户：

```mysql
mysql> CREATE USER repl_user IDENTIFIED BY 'BBDERS1@bbdops.com';
mysql> GRANT REPLICATION SLAVE,SUPER,RELOAD ON *.* TO repl_user;
```

REPLICATION SLAVE 权限也没啥，只是拥有这个权限的用户能够获取 master 上的二进制文件，但是最好还是单独为一个用户赋予这个权限，之后如果想禁止某些 slave 连接，只要删除该用户就可以了。



## 配置 slave

在 my.cnf 中加入以下配置：

```
server-id=2
relay-log=slave-relay-bin
relay-log-index=slave-relay-bin.index
```

relay 是中继的意思。



## 连接 master 和 slave

在 slave 上执行：

```mysql
mysql> CHANGE MASTER TO
   		     MASTER_HOST = 'drift-1',
    	     MASTER_PORT = 3306,
    	     MASTER_USER = 'repl_user',
    	     MASTER_PASSWORD = 'BBDERS1@bbdops.com';
mysql> START SLAVE;
```

查看状态：

```mysql
mysql> SHOW SLAVE STATUS;
```

在 mysql 8 上有一个错误：

```
Last_IO_Error: error connecting to master 'repl_user@drift-1:3306' - retry-time: 60 retries: 2 message: Authentication plugin 'caching_sha2_password' reported error: Authentication r
equires secure connection.
```

解决方案：

```mysql
mysql> STOP SLAVE;
mysql> CHANGE MASTER TO
   		     MASTER_HOST = 'drift-1',
    	     MASTER_PORT = 3306,
    	     MASTER_USER = 'repl_user',
    	     MASTER_PASSWORD = 'BBDERS1@bbdops.com',
    	     GET_MASTER_PUBLIC_KEY = 1;
mysql> START SLAVE;
```

加一个 GET_MASTER_PUBLIC_KEY = 1 的参数。

这样主从复制就大功告成了！！！



## 测试

在 master 上执行：

```mysql
mysql> CREATE DATABASE test;
mysql> USE test;
mysql> CREATE TABLE tbl (test TEXT);
mysql> INSERT INTO tbl VALUES ('Yeah! Replication!');
mysql> SELECT * FROM tbl;
mysql> FLUSH LOGS;
```

FLUSH LOGS 命令强制转换二进制日志，从而得到一个完整的二进制日志文件。

然后执行：

```
mysql> SHOW BINLOG EVENTS;
+-------------------+-----+----------------+-----------+-------------+-----------------------------------+
| Log_name          | Pos | Event_type     | Server_id | End_log_pos | Info                              |
+-------------------+-----+----------------+-----------+-------------+-----------------------------------+
| master-bin.000001 |   4 | Format_desc    |         1 |         125 | Server ver: 8.0.20, Binlog ver: 4 |
| master-bin.000001 | 125 | Previous_gtids |         1 |         156 |                                   |
| master-bin.000001 | 156 | Stop           |         1 |         179 |                                   |
+-------------------+-----+----------------+-----------+-------------+-----------------------------------+
3 rows in set (0.00 sec)
```

Log_name 是日志名字，Event_type 是事件类型，Server_id 是产生事件的服务器ID，Pos 是事件的开始位置，End_log_pos 是事件的结束位置。

这个命令默认只会列出 master-bi.000001 中的事件，更多的事件使用以下命令：

```mysql
mysql> SHOW BINLOG EVENTS IN 'master-bin.000003';
```

一个 binlog 会以一个 Rotate 事件结尾，并且 INFO 中记录了下一个文件的开始位置。

使用下面命令查看当前写入的是哪个二进制日志文件：

```mysql
mysql> SHOW MASTER STATUS;
```



## 删除 Slave

在 slave 上执行：

```mysql
mysql> STOP SLAVE;
mysql> RESET SLAVE;
mysql> DROP TABLE tbl;
```

在 master 上执行：

```mysql
mysql> RESET MASTER;
```

RESET SLAVE 删除了 slave 上复制用的所有文件，重新开始。RESET MASTER删除了所有的 binlog 并清空了 binlog 的索引文件。

必须要先 STOP SLAVE; 再执行之后的操作。



## 克隆 master

在 master 已经运行了很久了，这时再从头复制很不现实，所以要先备份，再进行复制！这里复制选择了最简单的 mysqldump，也可以有其他方式。

创建 master 备份，由于 maste 可能正在运行，而且缓存中有很多表，所以需要刷新所有表并锁定数据库，防止在检查 binlog 位置之前数据库发生改变：

```mysql
mysql> FLUSH TABLES WITH READ LOCK;
```

一旦数据库锁定就可以创建备份，并记录 binlog 位置了（注意这个命令行不要退出）：

```mysql
mysql> SHOW MASTER STATUS;
+-------------------+----------+--------------+------------------+-------------------+
| File              | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+-------------------+----------+--------------+------------------+-------------------+
| master-bin.000001 |      634 |              |                  |                   |
+-------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

记下这个位置后就可以创建备份了，在另外一个 master 命令行中执行：

```bash
$ mysqldump -uroot -pBBDERS1@bbdops.com --all-databases > backup.sql
```

在第一个命令行中解锁：

```mysql
mysql> UNLOCK TABLES;
```

接下来在 slave 中恢复：

```bash
$ mysql -uroot -pBBDERS1@bbdops.com < backup.sql
```

备份及恢复完成后，开始复制，在 slave 中执行：

```mysql
mysql> CHANGE MASTER TO
   		     MASTER_HOST = 'drift-1',
    	     MASTER_PORT = 3306,
    	     MASTER_USER = 'repl_user',
    	     MASTER_PASSWORD = 'BBDERS1@bbdops.com',
    	     GET_MASTER_PUBLIC_KEY = 1,
    	     MASTER_LOG_FILE='master-bin.000001',
    	     MASTER_LOG_POS = 634;
mysql> START SLAVE;
```





## 克隆 Slave

只要有一个 slave 连在 master 之上，就可以用这个 slave 创建新的 slave，就不需要离线 master 了。

克隆 slave 与 克隆 master 基本一致，只是在克隆时需要使用 `STOP SLAVE;` 命令停止 slave。













