# MySQL 离线环境主从搭建

系统为 CentOS 7。



## MySQL 安装

首先安装 MySQL：https://juejin.im/post/5d07cf13f265da1bd522cfb6

我使用 RPM 包安装，去官方下载 RedHat 版本的 MySQL rpm 包。下载地址：https://dev.mysql.com/downloads/mysql/

解压：

```bash
$ tar -vxf mysql-5.7.29-1.el7.x86_64.rpm-bundle.tar
```

查看系统中是否安装了 mariadb:

```bash
$ rpm -qa | grep mariadb
```

如果安装了就用 `rpm -e` 删除。

将解压后的几个 rpm 包复制到自建的 repo 目录中，可查看： [yum离线源的配置.md](../../Linux/yum/yum离线源的配置.md) 

然后：

```bash
$ yum install mysql-community-server --enablerepo=rpm
```

这会将服务端和客户端全部安装上。

配置文件：`/etc/my.cnf`，更改数据存放目录，我这里改为 `/mnt/vde/mysql`：

```bash
$ mkdir /mnt/vde/mysql
$ chown -R mysql:mysql /mnt/vde/mysql
```

启动：

```bash
$ systemctl start mysqld
```

查看状态：

```bash
$ systemctl status mysqld
```

查看初始密码：

```bash
$ grep 'temporary password'  /var/log/mysqld.log
```

登录：

````bash
$ mysql -u root -p
````

在使用前还需要修改密码：

```mysql
mysql> ALTER USER root@localhost identified by 'BBDERS1@bbdops.com';
```

密码组成是有规则的，这里学一下。

查看 `validate_password_policy` 值：

```mysql
mysql> select @@validate_password_policy;
+----------------------------+
| @@validate_password_policy |
+----------------------------+
| MEDIUM                     |
+----------------------------+
1 row in set (0.00 sec)
```

默认是 MEDIUM，所以刚开始设置的密码必须符合长度，且必须含有数字，小写或大写字母，特殊字符。

查看具体规则：

```mysql
mysql> SHOW VARIABLES LIKE 'validate_password%';
+--------------------------------------+--------+
| Variable_name                        | Value  |
+--------------------------------------+--------+
| validate_password_check_user_name    | OFF    |
| validate_password_dictionary_file    |        |
| validate_password_length             | 8      |
| validate_password_mixed_case_count   | 1      |
| validate_password_number_count       | 1      |
| validate_password_policy             | MEDIUM |
| validate_password_special_char_count | 1      |
+--------------------------------------+--------+
7 rows in set (0.00 sec)
```

`validate_password_check_user_name` 检查密码中是否有用户名

`validate_password_dictionary_file` 用于验证密码强度的字典文件路径。

`validate_password_length` 密码最小长度

`validate_password_mixed_case_count` 密码至少要包含的小写字母个数和大写字母个数。

`validate_password_number_count` 密码至少要包含的数字个数。

`validate_password_policy` 密码强度检查等级，0/LOW、1/MEDIUM、2/STRONG。有以下取值：

| Policy      | Tests Performed                                              |
| ----------- | ------------------------------------------------------------ |
| 0 or LOW    | Length                                                       |
| 1 or MEDIUM | Length; numeric, lowercase/uppercase, and special characters |
| 2 or STRONG | Length; numeric, lowercase/uppercase, and special characters; dictionary file |

修改密码策略配置：

```mysql
mysql> set global validate_password_length=10;
```



#### 修改默认端口号

mysql 默认占用 3306，在 mysql shell 内查看：

```mysql
mysql> show global variables like 'port';
```

在 `/etc/my.cnf` 中添加配置：

````
[mysqld]  
port=3506
````

重启：

````bash
$ systemctl restart mysqld
````

登录，本机不用加端口号，因为 mysql 客户端也会读取配置文件：

```bash
$ mysql -u root -p 
```

再次检查：

```mysql
mysql> show global variables like 'port';
```



## 配置主从

在另一台服务器上再按照上边的教程再安装一个实例。

#### 主服务器开启binlog

添加如下配置：

```
server-id=11
log-bin=node01-bin
binlog-format=ROW
binlog-row-image=minimal  #只记录要修改的行
log-slave-updates=true
gtid-mode=on
enforce-gtid-consistency=true
master-info-repository=TABLE
relay-log-info-repository=TABLE
sync-master-info=1
slave-parallel-workers=8
sync_binlog=1
binlog-checksum=CRC32
master-verify-checksum=1
slave-sql-verify-checksum=1
binlog-rows-query-log_events=1
#expire_logs_days=5
max_binlog_size=1024M
```

重启 mysqld 服务

#### 从服务器开启binlog

添加一下配置：

```bash
server-id=12
log-bin=node02-bin
binlog-format=ROW
binlog-row-image=minimal  #只记录要修改的行
log-slave-updates=true
gtid-mode=on
enforce-gtid-consistency=true
master-info-repository=TABLE
relay-log-info-repository=TABLE
sync-master-info=1
slave-parallel-workers=8
sync_binlog=1
binlog-checksum=CRC32
master-verify-checksum=1
slave-sql-verify-checksum=1
binlog-rows-query-log_events=1
#expire_logs_days=5
max_binlog_size=1024M
```

重启 mysqld 服务



#### 主服务器配置

```mysql
mysql> grant replication client,replication slave,super,reload  on *.* to 'root'@'fueltank-2' identified by 'BBDERS1@bbdops.com';
mysql> flush privileges;
```



#### 从服务器配置

```mysql
mysql> change master to
    -> master_host='fueltank-1',
    -> master_port=3306,
    -> master_user='root',
    -> master_password='BBDERS1@bbdops.com',
    -> master_auto_position=1;
mysql> start slave;
mysql> show slave status\G;
```

命令查看到如下两行，说明配置正确。
Slave_IO_Running: Yes
Slave_SQL_Running: Yes



### 测试

分别在两台服务器创建数据库和数据表，查看同步情况。
分别在两台服务器上删除数据库和数据表，查看同步情况。

我这里是完美的。



## 主从状态检查脚本

```bash
$ mkdir /opt/mysqlcheck/crontab/logs -p
```

编写 `/opt/mysqlcheck/check_mysql_slave.sh` ，写入以下内容：

```bash
#!/bin/sh
# check_mysql_slave status
# author sysit
ip=eth0  #网卡名称
mysql_binfile=/usr/bin/mysql
mysql_user=root  #MySQL数据库账号
mysql_pass=BBDERS1@bbdops.com  #密码
mysql_sockfile=/var/lib/mysql/mysql.sock
datetime=`date +"%Y-%m-%d/%H:%M:%S"`   #获取当前时间
mysql_slave_logfile=/opt/mysqlcheck/crontab/logs/check_mysql_slave.log   #日志文件路径，必须提前创建好
slave_ip=`172.20.20.179`
status=$($mysql_binfile -u$mysql_user -p$mysql_pass -S $mysql_sockfile -e "show slave status\G" | grep -i "running")
Slave_IO_Running=`echo $status | grep Slave_IO_Running | awk ' {print $2}'`
Slave_SQL_Running=`echo $status | grep Slave_SQL_Running | awk '{print $2}'`
if [ "$Slave_IO_Running" = "Yes" -a "$Slave_SQL_Running" = "Yes" ]
then
    echo " $datetime $slave_ip Slave is Running!" >> $mysql_slave_logfile
else
    echo " $datetime $slave_ip Slave is not running!" >> $mysql_slave_logfile
    #$mysql_binfile -u$mysql_user -p$mysql_pass -S $mysql_sockfile -e "STOP SLAVE;"
    #$mysql_binfile -u$mysql_user -p$mysql_pass -S $mysql_sockfile -e "SET GLOBAL SQL_SLAVE_SKIP_COUNTER=1;"
    #$mysql_binfile -u$mysql_user -p$mysql_pass -S $mysql_sockfile -e "START SLAVE;"
    #$mysql_binfile -u$mysql_user -p$mysql_pass -S $mysql_sockfile -e "EXIT"
fi
```









