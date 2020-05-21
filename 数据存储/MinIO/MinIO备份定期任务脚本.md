# MinIO备份定期任务脚本

这里需要两个脚本，定期删除脚本，和定期创建 bucket 脚本。

编辑定期删除脚本：

````bash
$ vim /opt/minio/minio-backup-rm.sh
````

内容如下

```bash
#!/bin/bash

DATE=`date -d '2 day ago' +'%Y-%m-%d'`

mc rm --recursive --force myminio/$DATE >> /opt/minio/minio-backup-rm.log 2>&1

echo "$DATE : minio remove bucket success" >> /opt/minio/minio-backup-rm.log
```

编辑定期创建 bucket 的脚本：

```bash
$ vim /opt/minio/minio-bucket-create.sh
```

内容如下：

```bash
#!/bin/bash

DATE=`date -d tomorrow +'%Y-%m-%d'`

mc mb myminio/$DATE >> /opt/minio/minio-bucket-create.log 2>&1

echo "$DATE : minio create myminio/$DATE success" >> /opt/minio/minio-bucket-create.log 
```



赋予执行权限：

```bash
$ chmod +x /opt/minio/minio-backup-rm.sh
$ chmod +x /opt/minio/minio-bucket-create.sh
```

定时任务，编辑 `/etc/corntab`，加入以下内容

```bash
0 1 * * * root /opt/minio/minio-bucket-create.sh
0 1 * * * root /opt/minio/minio-backup-rm.sh
```

重启 crond：

```bash
$ systemctl restart crond
```













