# Keepalived 入门教程

官方文档：https://www.keepalived.org/doc/

安装：

```bash
$ sudo yum install keepalived
```

启动：

```bash
$ sudo systemctl enable keepalived
$ sudo systemctl start keepalived
```

启动完成后，使用 `ip a` 来查看挂载到 eth0 上的 VIP。

配置文件是 `/etc/keepalived/keepalived.conf`

