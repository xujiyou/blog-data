---
title: Systemd管理Kafka
date: 2020-06-18 19:56:31
tags:
---

Kafka 版本：2.0.0

Scala 版本：2.12

安装目录：/opt/kafka

修改配置文件：

```bash
$ vim /opt/kafka/config/server.propertie
```

修改的配置如下：

````
log.dirs=/home/kafka-logs
log.retention.hours=72
log.retention.bytes=107374182400
````

数据保存3天。最大为 100 G，/home 目录容量为 165G。

编写 service 文件：

```
$ vim /usr/lib/systemd/system/kafka.service
```

内容如下：

```
[Unit]
Description=Apache Kafka server (broker)
Documentation=http://kafka.apache.org/documentation.html
Requires=network.target remote-fs.target
After=network.target remote-fs.target kafka-zookeeper.service

[Service]
Type=simple
User=kafka
Group=kafka
Environment=JAVA_HOME=/usr/java/jdk1.8.0_251-amd64
ExecStart=/opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/server.properties
ExecStop=/opt/kafka/bin/kafka-server-stop.sh

[Install]
WantedBy=multi-user.target
```

添加系统用户：

```bash
$ groupadd kafka --system
$ useradd -m kafka -g kafka --system
```

创建数据目录：

```bash
$ mkdir /home/kafka-logs
$ chown -R kafka:kafka /home/kafka-logs/
```

在 /etc/profile 添加环境变量：

```
export KAFKA_HOME=/opt/kafka
export PATH=$PATH:$ZOOKEEPER_HOME/bin:$KAFKA_HOME/bin
```

使之生效：

```bash
$ source /etc/profile
```

启动：

```bash
$ systemctl daemon-reload
$ systemctl enabel kafka
$ systemctl start kafka
```

查看状态：

```bash
$ systemctl status kafka
$ lsof -i:9092
```

测试：

```

```











