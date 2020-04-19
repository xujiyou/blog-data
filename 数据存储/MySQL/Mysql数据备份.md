# MySQL 数据备份

备份前先考虑数据量大小，及储存备份的数据盘的大小。

备份脚本：

```bash
#!/bin/bash

BACK_DIR="/data1/mysql-backup"
DATE=`date +'%F-%T'`

mysqldump -uroot -pMyPassword --all-databases --triggers --routines --events > $BACK_DIR/server05-mysql-CDH-$DATE.sql

rsync -a --delete $BACK_DIR root@server01:/data1/server05-mysql-CDH-backup/

find $BACK_DIR -name "*.sql" -mtime +7 | xargs rm -f

echo "$DATE backup done" >> $BACK_DIR/mysql-backup-CDH.log
```

如果没有 `rsync` 命令，可以安装：

```bash
$ sudo yum install rsync -y
```

脚本写完之后，先在控制台测试一下。



设置定时任务，修改 `/etc/crontab` 文件：

```
0 0 * * * root /data1/mysql-backup/mysql-backup.sh
```



重启定时任务后台进程：

```bash
$ sudo systemctl restart crond
```



