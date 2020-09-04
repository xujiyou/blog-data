# Docker 更换储存位置

最好将 Docker 的数据放到挂载盘里

1. 关闭docker服务

```bash
$ service docker stop
```

2. 将/var/lib/docker文件夹拷贝到指定目录

```bash
$ cp -rf /var/lib/docker /<目标路径>/
```

3. 建立软连接

```bash
$ sudo ln -s /mnt/vde/docker /var/lib/docker
```

4. 重启docker服务

```bash
$ service docker start
```



## 更改配置来切换储存位置

```bash
$ sudo vim /etc/docker/daemon.json 
{
  "graph": "/new-path/docker"
}
```

查看当前储存位置

```bash
$ sudo docker info | grep "Docker Root Dir"
```

