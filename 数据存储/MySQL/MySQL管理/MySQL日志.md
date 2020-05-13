# MySQL 日志

MySQL 分别有几种日志：

- 错误日志
- 一般的查询日志
- Binary log，储存数据更改的语句，也是基于这个日志做的主从复制
- Relay log，记录从主服务器那里获取到的数据
- 慢查询日志
- DDL日志（元数据日志）

查看日志输出格式：

```mysql
mysql> SHOW VARIABLES LIKE "log_output";
```

默认是 FILE。

查看查询日志的状态：

```mysql
mysql> SHOW VARIABLES LIKE "general_log";
```

默认是关闭的。

查看慢查询日志的状态：

````mysql
mysql> SHOW VARIABLES LIKE "slow_query_log";
````

默认也是关闭的。

查看这俩日志的日志格式：

```mysql
mysql> SHOW CREATE TABLE mysql.general_log;
mysql> SHOW CREATE TABLE mysql.slow_log;
```

开启查询日志和慢查询日志：

```mysql
mysql> SET PERSIST general_log='ON';
mysql> SET PERSIST slow_query_log='ON';
```

查看查询日志文件：

```mysql
mysql> SHOW VARIABLES LIKE "general_log_file";
```

查看慢查询日志文件：

```mysql
mysql> SHOW VARIABLES LIKE "slow_query_log_file";
```

刷新日志，重新记录：

````bash
$ mysqladmin flush-logs -u root -p
````



## 错误日志

如何开启错误日志那，需要在配置文件中加入以下配置：

```
[mysqld]
log-error=
```

不用填写值，MySQL 会默认在数据目录中生成一个以 .err 为后缀的错误日志文件。当然，这里也可以自定义文件名。

然后重启 MySQL 就可以看到错误日志文件了。

查看错误日志文件：

```mysql
mysql> SHOW VARIABLES LIKE 'log_error';
```

如果这里 log-error 未指定，则会打印到控制台，在 CentOS 中就是 `/var/log/mysqld.log`

#### 日志处理器

通过以下命令查看错误日志处理器：

```mysql
mysql> SELECT @@GLOBAL.log_error_services;
```

安装一个日志处理器：

```mysql
mysql> INSTALL COMPONENT 'file://component_log_sink_syseventlog';
mysql> SET PERSIST log_error_services = 'log_filter_internal; log_sink_internal; log_sink_syseventlog';
```

名为 `log_sink_syseventlog` 的处理器的作用是将错误日志写入到系统事件日志中，比如在 CentOS 中是 journal 中，使用这个处理器后，可以通过以下方式看到错误日志：

```bash
$ sudo journalctl -xeu mysqld
```



卸载刚刚安装的处理器：

```mysql
mysql> SET GLOBAL log_error_services = 'log_filter_internal; log_sink_internal';
mysql> UNINSTALL COMPONENT 'file://component_log_sink_syseventlog';
```



安装 JSON 日志处理器：

```mysql
mysql> INSTALL COMPONENT 'file://component_log_sink_json';
mysql> SET PERSIST log_error_services = 'log_filter_internal; log_sink_internal; log_sink_json';
```

这样设置之后就可以在数据储存目录中看到一个 json 文件了，比如我的：

![image-20200513210058132](../../../resource/image-20200513210058132.png)

关于日志格式，可以查看：https://dev.mysql.com/doc/refman/8.0/en/error-log-format.html

