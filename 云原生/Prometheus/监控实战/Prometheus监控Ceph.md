# Prometheus 监控 Ceph

新版本的 Ceph 原生支持 Prometheus。

开启 ：

```bash
$ sudo ceph mgr module enable prometheus --force
```

查看端口：

```bash
$ sudo netstat -nltp | grep mgr
tcp6       0      0 :::9283                 :::*                    LISTEN      51889/ceph-mgr
```

测试访问：

```bash
$ curl 127.0.0.1:9283/metrics
```

添加 prometheus 配置：

```bash
  - job_name: 'ceph'
    honor_labels: true
    static_configs:
     - targets:
        - drift-1:9283
       labels:
         instance: ceph
```

重启 Prometheus ：

```bash
$ sudo systemctl restart prometheus
```

