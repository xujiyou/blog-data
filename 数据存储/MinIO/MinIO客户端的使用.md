# MinIO客户端的使用

MinIO 的客户端是一个名为 mc 的二进制文件。

下面是一些常用操作。

首先配置 mc，在每一台需要备份的机子上操作：

```bash
$ mc config host add myminio http://192.168.1.18:9000 test test@test.com
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

查看桶列表：

```bash
$ mc ls myminio
```

删除桶中的全部文件：

```bash
$ mc rm --recursive --force myminio/one-backup
```



删除桶：

```bash
$ mc rm --recursive --force myminio/one-backup
```





## MinIO 永久链接

将一个 bucket 设置为 public ，其内的对象就可以获得永久链接了，这里将 test 的桶设置为 public 的：

```bash
$ mc policy set public myminio/test
```

这样就可以通过 url 直接获取了，而且三个点都可以提供服务，格式是 `http://192.168.6.124:29000/bucket/object` ，比如：

```bash
$ wget http://192.168.6.124:29000/test/download.png
$ wget http://192.168.6.125:29000/test/download.png
$ wget http://192.168.6.126:29000/test/download.png
```

然后再通过 Nginx 代理一下就可以了。













