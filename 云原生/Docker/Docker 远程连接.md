# Docker 远程连接

服务端是 CentOS 系统，客户端是 MacOS。

首先在服务端编辑 `/etc/docker/daemon.json` ，加入：

```
"hosts": [
    "tcp://0.0.0.0:2376",
    "unix:///var/run/docker.sock"
]
```

然后执行 `sudo systemctl edit docker.service` 加入以下代码：

```
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd
```

完成后重启Docker：

```
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker.service
```

然后在客户端执行以下命令来测试是否成功：

```
$ docker -H tcp://fueltank-1:2376 info
```

