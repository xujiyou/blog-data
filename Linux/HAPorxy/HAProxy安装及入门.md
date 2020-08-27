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



## 配置文件内容

```ini
#---------------------------------------------------------------------
# Example configuration for a possible web application.  See the
# full configuration options online.
#
#   http://haproxy.1wt.eu/download/1.4/doc/configuration.txt
#
#---------------------------------------------------------------------

#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    # to have these messages end up in /var/log/haproxy.log you will
    # need to:
    #
    # 1) configure syslog to accept network log events.  This is done
    #    by adding the '-r' option to the SYSLOGD_OPTIONS in
    #    /etc/sysconfig/syslog
    #
    # 2) configure local2 events to go to the /var/log/haproxy.log
    #   file. A line like the following can be added to
    #   /etc/sysconfig/syslog
    #
    #    local2.*                       /var/log/haproxy.log
    #
    log         127.0.0.1 local2

    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000

listen kubernetes
    mode tcp
    balance roundrobin

    # nginx
    bind *:80
    bind *:443

    # mysql 8.0
    bind *:3306

    # mysql 5.7
    bind *:3307

    # redis 5.0
    bind *:6379

    # elasticsearch 7.6
    bind *:9200
    bind *:9300

    # elasticsearch 6.8
    bind *:9201
    bind *:9301

    # nacos 1.2
    bind *:8848

    # zookeeper 3.6
    bind *:2181

    # kafka 2.4
    bind *:9092

    # fastdfs 5.12
    bind *:22122
    bind *:11411

    # vsftpd 3.0
    bind *:21
    bind *:21100-21110

    # mongodb 4.2
    bind *:27017

    # postgresql 12.3
    bind *:5432

    # red alert
    bind *:30010-30029

    # canghai
    bind *:30030-30049

    server test-kubenode-1 test-kubenode-1.test.bbdops.com check
    server test-kubenode-2 test-kubenode-2.test.bbdops.com check
    server test-kubenode-3 test-kubenode-3.test.bbdops.com check
    server test-kubenode-4 test-kubenode-4.test.bbdops.com check
    
listen ceph
    mode tcp
    balance roundrobin

    bind *:8443
    bind *:9283

    server test-kubenode-1 test-kubenode-1.test.bbdops.com check
```

