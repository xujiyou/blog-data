# Syslog-ng 使用教程

syslog-ng 是 syslog 的升级版

首先安装：

```
sudo yum install syslog-ng
```

启动：

```
sudo systemctl start syslog-ng
```

查看状态：

```
sudo systemctl status syslog-ng
```

配置文件为：`/etc/syslog-ng/syslog-ng.conf`

syslog-ng 使用 UDP 的 514 端口，使用 `netstat -aun` 可以查看端口占用

官方教程在这里：https://www.syslog-ng.com/technical-documents/doc/syslog-ng-open-source-edition/3.25/administration-guide

