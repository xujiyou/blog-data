# MySQL 配置

MySQL 的配置非常众多，可以通过以下命令查看：

```bash
$ mysqld --verbose --help
```

以上命令是会带上文件中的配置值的，如果不想看自己配置的值，只想看系统的默认值，可以使用以下命令：

```bash
$ mysqld --no-defaults --verbose --help
```

关于 mysqld 的完整的选项列表请看：https://dev.mysql.com/doc/refman/8.0/en/server-options.html

也可以使用以下命令查看运行中使用到的配置：

```mysql
mysql> SHOW VARIABLES;
mysql> SHOW VARIABLES LIKE 'datadir'
```

这些配置也可以通过下面的命令查看：

```bash
$ mysqladmin variables -u root -p
```

完整的变量信息，查看：https://dev.mysql.com/doc/refman/8.0/en/server-system-variable-reference.html

可以使用以下命令查看运行中的统计和状态信息：

```mysql
mysql> SHOW STATUS;
```

等同于：

```bash
$ mysqladmin extended-status -u root -p
```

统计信息的完整列表可见：https://dev.mysql.com/doc/refman/8.0/en/server-status-variable-reference.html





## 配置验证

验证配置信息，如果不加任何其他参数，默认会验证 `/etc/my.cnf`：

```bash
$ mysqld --validate-config
```

验证某个配置对不对，可以这样：

```bash
$ sudo mysqld --validate-config --log_error_verbosity=2 --read-only=s --transaction_read_only=true
```

这里会有以下输出：

```
2020-05-13T06:22:59.183405Z 0 [Warning] [MY-000076] [Server] option 'read_only': boolean value 's' was not recognized. Set to OFF.
```

这个命令会验证配置是否正确，如果正确的话就不会有输出。

验证指定的配置文件：

```bash
$ mysqld --defaults-file=/etc/my.cnf --validate-config
```

在 MySQL 8.0 中，如果在 `/etc/my.cnf` 加入以下配置：

```
tx_read_only=ON
```

再进行验证配置：

```bash
$ mysqld --validate-config
```

会出现以下错误：

```
2020-05-13T06:26:29.426288Z 0 [ERROR] [MY-000067] [Server] unknown variable 'tx_read_only=ON'.
2020-05-13T06:26:29.426396Z 0 [ERROR] [MY-010119] [Server] Aborting
```



## 指定配置

可以使用以下方式指定配置：

```bash
$ mysqld --innodb-log-file-size=16M --max-allowed-packet=1G
```

但是我不太喜欢这种方式，我还是喜欢在配置文件里面修改配置，CentOS 中使用 YUM 安装的配置文件为 `/etc/my.cnf`：

```
[mysqld]
innodb_log_file_size=16M
max_allowed_packet=1G
```

许多系统变量都是动态的，可以在运行时通过使用[`SET`](https://dev.mysql.com/doc/refman/8.0/en/set-variable.html) 语句来更改 。下面来测试动态更改：

```mysql
mysql> SET GLOBAL max_connections = 1000;
mysql> SET @@GLOBAL.max_connections = 1000;
```

查看被修改了的变量：

```mysql
mysql> SHOW VARIABLES LIKE 'max_connections';
```

不过这时候虽然更改了配置，但是一旦 mysqld 重启，就会恢复到以前的配置。

将全局系统变量持久化到 `mysqld-auto.cnf`文件：

```mysql
mysql> SET PERSIST max_connections = 1000;
mysql> SET @@PERSIST.max_connections = 1000;
```

如果执行了 SET GLOBAL ，也可以将全局系统变量持久化到 `mysqld-auto.cnf`文件（不设置运行时值）：

```mysql
mysql> SET PERSIST_ONLY back_log = 1000;
mysql> SET @@PERSIST_ONLY.back_log = 1000;
```

这样，即使重启了，也不会耽误修改配置的生效

`mysqld-auto.cnf` 放在 mysql 数据储存目录了，默认是 `/var/lib/mysql/mysqld-auto.cnf` 。

内容如下：

```json
{
    "Version": 1,
    "mysql_server": {
        "max_connections": {
            "Value": "1000",
            "Metadata": {
                "Timestamp": 1589356107051523,
                "User": "root",
                "Host": "localhost"
            }
        }
    }
}
```











