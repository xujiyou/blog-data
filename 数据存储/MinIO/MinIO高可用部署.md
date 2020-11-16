# MinIO高可用部署



## 高可用部署

MinIO 的高可用模式最少为四块盘，并且这四块盘需要是空白的！！！

把盘挂载到文件系统，然后在各主机下都执行以下脚本：

```bash
#!/bin/bash

export MINIO_ACCESS_KEY=admin
export MINIO_SECRET_KEY=********
minio server http://fueltank-1:9000/data1 http://fueltank-1:9000/data2 http://fueltank-2:9000/data1 http://fueltank-3:9000/data1
```



## 添加 https 支持

如果使用 root 账户启动的：

```bash
sudo cp server.crt /root/.minio/certs/public.crt
sudo cp server.key /root/.minio/certs/private.key
```

然后启动脚本这样：

```bash
#!/bin/bash

export MINIO_ACCESS_KEY=admin
export MINIO_SECRET_KEY=********
minio server https://fueltank-1:9000/data1 https://fueltank-1:9000/data2 https://fueltank-2:9000/data1 https://fueltank-3:9000/data1
```

## 坑点

>掉点：
>
>当一个节点宕机后（宕机节点数少于等于MinIO节点的一半），MinIO 可以保证数据完整，但是这时MinIO只是可读，不可写！！！

MinIO 看上去挺好用，其实还是很坑的！！！

MinIO 扩展并不方便，只能成倍扩展！！！