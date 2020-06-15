# CoreDNS 使用 etcd 插件

首先安装 etcd，安装过程略过，教程在：  [Kubernetes二进制安装.md](../Kubernetes/Kubernetes二进制安装.md) ，为简单起见，这里安装一个单点。



## 搭建 CoreDNS

下载 CoreDNS 的二进制文件，并将它放到 PATH 中。

编写 service 文件：

```bash
$ sudo vim /usr/lib/systemd/system/coredns.service
```

内容如下：

```
[Unit]
Description=Coredns
After=network-online.target
    
[Service]
Type=simple
ExecStart=/usr/bin/coredns -conf /etc/coredns/Corefile
Restart=always
ExecStop=/bin/kill -9 $MAINPID
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=coredns
    
    
[Install]
WantedBy=multi-user.target
```

创建配置文件：

```bash
$ sudo mkdir /etc/coredns
$ sudo vim /etc/coredns/Corefile
```

内容如下：

```
.:53 {
    # prometheus :9153    # prometheus url 监控插件 http://host:9153/metrics
    
    etcd {
        stubzones
        fallthrough     # 把ectd无法解析的子域名交给proxy
        path /drift #注意以 etcd 的键要以这个东西开头
        endpoint http://172.20.20.110:2379
        upstream /etc/resolv.conf
    }
    errors
    whoami
    log
    cache 30    # 缓存域名的解析300s
    proxy . /etc/resolv.conf
}
```

启动 CoreDNS：

```bash
$ sudo systemctl start coredns
```

测试：

```bash
$ etcdctl put /drift/com/drift-444/www '{"host":"172.20.20.110"}'
$ dig @172.20.20.110 www.drift-444.com
```

修改 `/etc/resolv.conf` ，将以下内容放在第一行：

```
nameserver 172.20.20.110
```

然后：

```bash
ping www.drift-444.com
```

大功告成。









