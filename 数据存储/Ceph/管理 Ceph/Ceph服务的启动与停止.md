# Ceph 服务的启动与停止

官方文档：https://ceph.readthedocs.io/en/latest/rados/operations/operating/

主要看 systemd 是怎么管理服务的！



## 启动

启动所有 Ceph 守护进程：

```bash
$ sudo systemctl start ceph.target
```

根据守护进程类型启动：

```bash
$ sudo systemctl start ceph-osd.target
$ sudo systemctl start ceph-mon.target
$ sudo systemctl start ceph-mds.target
```



## 停止

停止所有 Ceph 守护进程：

```bash
$ sudo systemctl stop ceph\*.service ceph\*.target
```

根据守护进程类型停止：

```bash
$ sudo systemctl stop ceph-mon\*.service ceph-mon.target
$ sudo systemctl stop ceph-osd\*.service ceph-osd.target
$ sudo systemctl stop ceph-mds\*.service ceph-mds.target
```

停止某个进程：

```
sudo systemctl stop ceph-osd@{id}
sudo systemctl stop ceph-mon@{hostname}
sudo systemctl stop ceph-mds@{hostname}
```

例如：

```bash
$ sudo systemctl stop ceph-osd@1
$ sudo systemctl stop ceph-mon@ceph-server
$ sudo systemctl stop ceph-mds@ceph-server
```



## 查看状态

查看所有 Ceph 守护进程的状态：

```bash
$ sudo systemctl status ceph.target
```

列出所有 Ceph 守护进程的状态：

```bash
$ sudo systemctl status ceph\*.service
```

