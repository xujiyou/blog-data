---
title: Nginx编译安装
date: 2020-07-20 19:07:25
tags:
---

Nginx 官方文档：http://nginx.org/en/docs/

二进制安装文档：http://nginx.org/en/docs/configure.html

最新标准版为 1.18，下载地址是：http://nginx.org/en/download.html



## 编译

安装编译环境：

```bash
$ yum -y install gcc pcre pcre-devel zlib zlib-devel openssl openssl-devel
```

创建用户：

```bash
$ useradd -r nginx
```

配置：

```
$ ./configure \
    --user=root \
    --group=root \
    --prefix=/usr/local/nginx/ \
    --sbin-path=/usr/bin/nginx \
    --conf-path=/etc/nginx/nginx.conf \
    --error-log-path=/var/log/nginx/error.log \
    --http-log-path=/var/log/nginx/access.log \
    --pid-path=/var/run/nginx/nginx.pid \
    --lock-path=/var/run/nginx/nginx.lock \
    --http-client-body-temp-path=/var/lib/nginx/client_body_temp \
    --http-proxy-temp-path=/var/lib/nginx/proxy_temp \
    --http-fastcgi-temp-path=/var/lib/nginx/fastcgi_temp \
    --http-uwsgi-temp-path=/var/lib/nginx/uwsgi_temp \
    --http-scgi-temp-path=/var/lib/nginx/scgi_temp \
    --with-http_ssl_module
```

编译：

```bash
$ make
```

安装：

```bash
$ make install
```

验证：

````bash
$ nginx -V
````

返回了安装时配置的参数，这些参数配置是卸载 Nginx 时的依据。



## 启动

创建目录并赋予权限：

```bash
$ sudo mkdir /var/lib/nginx
$ sudo chown -R nginx:nginx /etc/nginx/
$ sudo chown -R nginx:nginx /var/lib/nginx
$ sudo chown -R nginx:nginx /var/run/nginx
$ sudo chown -R nginx:nginx /var/log/nginx
```

创建 `/usr/lib/systemd/system/nginx.service` ：

```
[Unit]
Description=The nginx HTTP and reverse proxy server
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/var/run/nginx/nginx.pid
# Nginx will fail to start if /run/nginx.pid already exists but has the wrong
# SELinux context. This might happen when running `nginx -t` from the cmdline.
# https://bugzilla.redhat.com/show_bug.cgi?id=1268621
ExecStartPre=/usr/bin/rm -f /var/run/nginx/nginx.pid
ExecStartPre=/usr/bin/nginx -t
ExecStart=/usr/bin/nginx
ExecReload=/bin/kill -s HUP $MAINPID
KillSignal=SIGQUIT
TimeoutStopSec=5
KillMode=process
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

启动 Nginx：

```bash
$ sudo systemctl enable nginx
$ sudo systemctl start nginx
```



## 总结

上面这套步骤和  yum 安装的没有很大差别，源码安装的好处在于可以自定义，可以灵活选择版本、模块。





