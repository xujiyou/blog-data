# MySQL 定期备份并传输到 MinIO



## server05 MySQL 元数据备份

server05 上的数据只是存放的 CDH 元数据，所以比较简单，编辑脚本：

```bash
$ vim /data1/mysq-backup/mysql-backup.sh
```

内容如下：

```bash
#!/bin/bash

BACK_DIR="/data1/mysql-backup"
DATE=`date +'%F-%T'`

mysqldump -uroot -pMyPassword --all-databases --triggers --routines --events > $BACK_DIR/server05-mysql-CDH-$DATE.sql

mc cp $BACK_DIR/server05-mysql-CDH-$DATE.sql myminio/$DATE

find $BACK_DIR -name "*.sql" -mtime +7 | xargs rm -f

echo "$DATE backup done" >> $BACK_DIR/mysql-backup-CDH.log
```

赋予执行权限：

```bash
$ chmod +x /data1/mysq-backup/mysql-backup.sh
```

定期任务脚本：

```bash
0 2 * * * root /data1/mysq-backup/mysql-backup.sh
```

重启 crond：

```bash
$ systemctl restart crond
```





## Server02 MySQL 数据备份

由于这台 MySQL 数据量较大，使用 mysqldump 比较恼火，这里使用到了一个叫 `innobackupex` 的命令来备份，这个命令不会锁表。

首先安装这个命令，去网上下载名为 `percona-xtrabackup-24-2.4.20-1.el7.x86_64` 的包，然后离线使用 `rpm -Uvh` 来安装即可，由于是离线环境，可能还需要自己去网上下载其他依赖包，比如我这里需要下载如下包：

```bash
$ wget https://www.percona.com/downloads/Percona-XtraDB-Cluster/5.5.37-25.10/RPM/rhel6/x86_64/Percona-XtraDB-Cluster-shared-55-5.5.37-25.10.756.el6.x86_64.rpm
$ wget http://mirror.centos.org/centos/7/os/x86_64/Packages/perl-Module-Install-1.06-4.el7.noarch.rpm
```

我这里装外这俩依赖才能安装 `percona-xtrabackup` 。

安装完成后，监测：

```bash
$ which xtrabackup
$ which innobackupex
```

数据导出命令：

```bash
$ innobackupex --defaults-fie=/etc/my.cnf -S /var/lib/mysql/mysql.sock --user=root --password='******' --throottle=2000 --stream=tar /data3/mysql-backup/ | gzip > /data3/mysql-backup/server02-mysql-backup.tar.gz
```

我这里导出花了 27 分钟，因为压缩占了很大的时间，压缩之后的数据是 11G。解压缩之后的数据是 39 G



下面编写完成的数据备份脚本：

```bash
$ vim /data3/mysql-backup/server02-mysql-backup.sh
```

内容如下：

```bash
#!/bin/bash

BACK_DIR="/data3/mysql-backup"
DATE=`date +'%F-%T'`

innobackupex --defaults-fie=/etc/my.cnf -S /var/lib/mysql/mysql.sock --user=root --password='******' --throottle=2000 --stream=tar $BACK_DIR | gzip > $BACK_DIR/server02-mysql-backup-$DATE.tar.gz

mc cp $BACK_DIR/server02-mysql-backup-$DATE.tar.gz myminio/$DATE

find $BACK_DIR -name "*.tar.gz" -mtime +3 | xargs rm -f

echo "$DATE backup done" >> $BACK_DIR/mysql-backup.log
```

定期任务脚本：

```
0 2 * * * root /data3/mysql-backup/server02-mysql-backup.sh
```





