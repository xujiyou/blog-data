# MinIO高可用部署

MinIO 的高可用模式最少为四块盘，并且这四块盘需要是空白的！！！

把盘挂载到文件系统，然后在各主机下都执行以下脚本：

```bash
#!/bin/bash

export MINIO_ACCESS_KEY=admin
export MINIO_SECRET_KEY=********
minio server http://fueltank-1:9000/data1 http://fueltank-1:9000/data2 http://fueltank-2:9000/data1 http://fueltank-3:9000/data1
```

