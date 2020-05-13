# mysql-shell 的使用

mysql-shell 并不和 MySQL Client 命令行一样。

下载地址：https://dev.mysql.com/downloads/shell/

我下载的是 macos 的 MySQL Shell 8.0.20 版本。

在 MySQL Shell 中，并不是使用的 SQL 语句，而是类似 MongoDB Shell 那样的 js 语句。

就目前的情况来说mysqlsh是一个数据库初学者的工具(会javascript,python不太精通SQL)，像资深的DBA应该还是用不太着的。

我个人感觉mysqlsh对一个dba来说并没有mysql这个客户端工具来的方便。

连接数据库：

```bash
$ mysqlsh root@fueltank-3:3306
```

