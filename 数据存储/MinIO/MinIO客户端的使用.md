# MinIO客户端的使用

MinIO 的客户端是一个名为 mc 的二进制文件。

下面是一些常用操作。

首先配置 mc：

```bash
$ mc config host add myminio http://192.168.1.18:9000 bbders bbders@bbdops.com
```

创建一个桶：

```bash
$ mc mb myminio/one-backup
```

创建一个文件：

```bash
$ echo "hello world" >> abc.txt
```

将文件放入桶中：

```bash
$ mc cp abc.txt myminio/one-backup
```

查看桶中的文件：

```bash
$ mc ls myminio
$ mc ls myminio/one-backup
```

删除桶：

```bash
$ mc rm --recursive --force myminio/one-backup
```

















