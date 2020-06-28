---
title: Binlog详解
date: 2020-06-16 16:23:19
tags:
---



参考《高可用MySQL（第2版）》第四章



## binlog 日志结构

binlog 包含若干二进制文件，另外还包含一个索引文件，这个索引记录了所有的二进制文件。

每个 binlog 文件包含若干个事件，每个文件都是以 Format_description 开始和 Rotate 事件结尾。注意，如果服务器突然宕机，binlog 文件结尾就不会是 Rotate 事件。

Format_description 事件包含了服务器信息，以及关于文件状态的关键信息。在服务器重启后，会在一个新的文件内写入一个新的 Format_description 事件，因为服务器重启后，服务器配置可能更新。

写完 binlog 文件后，会在文件末尾添加一个 Rotate 事件，该事件记录了下一个 binlog 文件，并且记录了文件的起始位置。

除了这俩控制事件，其他事件都被分成组，每个组大致对应一个事务，对于非事务引擎来说，一个组就是一个语句。

一个组要么全执行，要么全不执行。



## binlog 事件结构

每个事件由以下部分组成：

- **Common header** 包含基本信息，最重要的是事件类型和事件大小
- **Post header** 提交头与特定的事件类型有关。
- **Event body** 事件体，储存事件的主要数据。
- **Checksum** 校验和，是一个 32 位整数，用于检查事件写入后是否有损坏，5.6 版本加入。

其中 ，Common header 与 Post header 的大小是固定的，大小记录在 Format_description 事件之中。

Format_description 包含的信息如下：

- 服务器版本
- binlog 文件格式版本
- 通用头长度
- 提交头长度



## 事件校验

有三个配置会影响事件的校验和，分别是：

- **Bingo-checksum=type** CRC32 或 NONE，为 NONE 时关闭校验和，默认为 CRC32，生成校验和
- **master-verify-checksum=boolean** 在master 传给 slave 时，master 的线程是否要进行校验和验证；`SHOW BINLOG EVENTS` 也会用到这一项配置，默认关闭。
- **slave-sql-verify-checksum=boolean** 在 slave 数据库上应用事件之前，是否验证事件的校验和，默认关闭。

校验 binlog 文件：

```bash
$ sudo mysqlbinlog --verify-binlog-checksum /var/lib/mysql/master-bin.000001
```



## 将语句写入日志

MySQL 也支持基于行的复制。

在基于语句的复制中，实际执行的语句连同某些执行信息将一起被写入二进制日志。

二进制日志写入是有互斥锁的，只允许一个线程拥有锁。



## binlog 过滤器

有两个配置可以设置要记录二进制日志的库：

- binlog-do-db
- binlog-ignore-db

这俩选项可以多次使用，比如：

```
[mysqld]
binlog-ignore-db=one_db
binlog-ignore-db=two_db
```

在使用基于语句的过滤时，要千万注意，要使用 `USE db` 这样选定数据库，一定不要在语句中另外插入数据库名，否则会因为过滤在复制时造成意想不到的错误。



## 基于行的复制

基于语句的复制的历史比较长，考虑的情况比较多，有些特殊情况，基于语句的复制是处理不了的。

基于行的复制就是复制纳盒插入表中的真实数据。

基于行的复制不会复制产生变更的语句，而是复制每个被插入、删除或更新的行，所以不用考虑很多复杂的 SQL 语句，只需简单的考虑数据本身即可。

但是基于行的复制并不是全部都是优点，需要考虑的是：语句是否会大量更新行？还是只做简单的更新或插入？

如果语句要更新大量的行，那么基于语句的复制更快，因为语句相比行要少得多，但是如果语句很复杂，那基于行的复制可能就更快。

如果哦语句只更新少量的行，则基于行的复制更快。

如果想开启基于行的复制，需要在 MySQL 配置中加入以下配置：

```
binlog-format=ROW
```

默认参数是 STATEMENT，基于语句的复制：

```
binlog-format=STATEMENT
```



## 混合模式

通过以下配置来打开：

```
binlog-format=MIXED
```

这表示使用混合模式，是官方推荐的方式，原理很简单：正常情况下使用基于基于的复制，对不安全的语句则切换到基于行的复制。



## 清除 binlog 文件

通过设置下面的参数可以让服务器自动清除旧的 binlog 文件：

```
expire-logs-days=3
```

若要手工删除，可以使用以下命令：

```mysql
mysql> PURGE BINARY LOGS BEFORE '2020-06-29 00:00:00';
```

或者：

```mysql
mysql> PURGE BINARY LOGS TO 'master-bin.000002';
```





