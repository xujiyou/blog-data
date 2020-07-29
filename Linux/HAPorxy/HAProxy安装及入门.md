# HAProxy 安装及入门

HAProxy 是一个代理，像 Nginx 和 Envoy 一样，都可以在 L7 层代理。

安装很简单，只需：

```bash
$ sudo yum -y install haproxy
```

查看版本：

```bash
$ haproxy -v
```

HAProxy 的配置文件是 `/etc/haproxy/haproxy.cfg`

启动：

```bash
$ sudo systemctl enable haproxy
$ sudo systemctl start haproxy
```

查看状态：

```bash
$ sudo systemctl status haproxy
```



