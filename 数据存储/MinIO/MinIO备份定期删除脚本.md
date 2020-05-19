# MinIO备份定期删除脚本

编辑脚本：

````
$ vim /opt/minio/minio-backup-rm.sh
````

内容如下

```bash
#!/bin/bash

DATE=`date -d '3 day ago' +'%Y-%m-%d'`

mc rm --recursive --force myminio/$DATE >> /opt/minio/minio-backup-rm.log 2>&1

echo "$DATE : minio remove bucket success" >> /opt/minio/minio-backup-rm.log
```

赋予执行权限：

```
$ chmod +x /opt/minio/minio-backup-rm.sh
```

定时任务，编辑 `/etc/corntab`，加入以下内容

```bash
0 0 * * * root /opt/minio/minio-backup-rm.sh
```

重启 crond：

```bash
$ systemctl restart crond
```

