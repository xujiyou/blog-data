# Docker 更换储存位置

最好将 Docker 的数据放到挂载盘里

1.关闭docker服务

```
service docker stop
```

将/var/lib/docker文件夹拷贝到指定目录

```
cd /var/lib
cp -rf /docker /<目标路径>/
```

3.建立软连接

```
sudo ln -s /mnt/vde/docker/ /var/lib/
```

4.重启docker服务

```
service docker start
```

