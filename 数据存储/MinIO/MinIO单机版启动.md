# Minio 单机版启动

MinIO 官方只提供了一个 minio 二进制文件，需要手动写启动脚本和 service 文件。

创建启动脚本：

```bash
$ vim /opt/minio/run.sh
```

内容如下：

```bash
#!/bin/bash
export MINIO_ACCESS_KEY=bbders
export MINIO_SECRET_KEY=bbders@bbdops.com
/usr/bin/minio server /store/minio
```

赋予执行权限：

```bash
$ chmod +x /opt/minio/run.sh
```

创建 service 文件：

```bash
$ vim /usr/lib/systemd/system/minio.service
```

内容如下：

```ini
[Unit]
Description=Minio service
Documentation=https://docs.minio.io/
Wants=network-online.target
After=network-online.target

[Service]
User=root
Group=root
WorkingDirectory=/opt/minio/
ExecStart=/opt/minio/run.sh

Restart=on-failure
RestartSec=5

LimitNOFILE=65536

TimeoutStopSec=infinity
SendSIGKILL=no

[Install]
WantedBy=multi-user.target
```

开启启动：

```bash
$ systemctl enable minio
```

启动服务：

```bash
$ systemctl start minio
```

打开 Web 界面：http://192.168.1.18:9000

用户名为 bbders，密码为 bbders@bbdops.com













