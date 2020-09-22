# Ambari

Ambari 是 HDP 的管理端，分为 ambari-server 和 ambari-agent 。



## Ambari-Server

配置好 ambari 的源，比如：

```
[ambari]
name=Ambari 2.7.5
baseUrl=http://mirrors.bbdops.com/list/hortonworks/ambari-2.7.5.0/centos7/2.7.5.0-72
enabled=1
gpgcheck=0
```

安装 ambari-server：

```bash
$ yum install ambari-server -y
```

Ambari-Server 主要关注 4 个目录：

- 配置目录（/etc/ambari-server/conf）
- 日志目录（/var/log/ambari-server）
- Hadoop 服务组件目录（/usr/hdp）通过 Ambari 安装的组件会放在这个目录下。
- Ambari 服务目录（/usr/lib/ambari-server）Ambari 自身的服务会安装在这个目录。

安装完成后，要先配置：

```bash
$ ambari-server setup
```

启动 ambari-server：

```bash
$ ambari-server start
```

























