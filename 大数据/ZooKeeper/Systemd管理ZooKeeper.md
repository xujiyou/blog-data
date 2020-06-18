---
title: Systemd管理ZooKeeper
date: 2020-06-18 19:42:44
tags:
---

ZooKeeper 版本：3.4.13

ZooKeeper 安装位置：/opt/zookeeper

修改配置文件：

```bash
$ mv /opt/zookeeper/conf/zoo_sample.cfg /opt/zookeeper/conf/zoo.cfg
```

配置如下：

```
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/home/zookeeper
clientPort=2181
```

编写 service 文件：

```
$ vim /usr/lib/systemd/system/zookeeper.service
```

内容如下：

````
[Unit]
Description=ZooKeeper Service
Documentation=http://zookeeper.apache.org
Requires=network.target
After=network.target

[Service]
Type=forking
User=zookeeper
Group=zookeeper
ExecStart=/opt/zookeeper/bin/zkServer.sh start /opt/zookeeper/conf/zoo.cfg
ExecStop=/opt/zookeeper/bin/zkServer.sh stop /opt/zookeeper/conf/zoo.cfg
ExecReload=/opt/zookeeper/bin/zkServer.sh restart /opt/zookeeper/conf/zoo.cfg
WorkingDirectory=/home/zookeeper

[Install]
WantedBy=default.target
````

添加用户：

```bash
$ groupadd zookeeper --system
$ useradd -m zookeeper -g zookeeper --system
```

创建目录：

```
$ mkdir /home/zookeeper
$  chown -R zookeeper:zookeeper /home/zookeeper/
```

在 `/etc/profile` 添加环境变量：

````
export JAVA_HOME=/usr/java/jdk1.8.0_251-amd64
export ZOOKEEPER_HOME=/opt/zookeeper
export PATH=$PATH:$ZOOKEEPER_HOME/bin
````

使之生效：

````
source /etc/profile
````

启动：

```bash
$ systemctl daemon-reload
$ systemctl enable zookeeper
$ systemctl start zookeeper
```

查看状态：

```
$ systemctl status zookeeper
$ sudo lsof -i:2181
$ zkCli.sh
```







