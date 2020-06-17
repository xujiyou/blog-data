---
title: Broker配置
date: 2020-06-17 20:33:00
tags:
---

官方文档：https://kafka.apache.org/documentation/#brokerconfigs

broker 配置全部在 `conf/server.properties` 内。

最基本的配置如下：

```
broker.id
log.dirs
zookeeper.connect
```

分别制定了 broker 的 id ，数据储存的地址，和 zk 地址。



#### advertised.listeners

`advertised_listeners` 监听器会注册在 `zookeeper` 中；

这个配置定义外部需要访问的地址。

如果没有设置，默认跟 listeners 的配置一样，一般用于 IaaS 或 Docker 环境中，如果不需要外网连接就不需要设置，与 listeners 不同的是，这个配置不允许设置为 0.0.0.0



#### auto.create.topics.enable

在服务器上开启自动创建 topic，默认为 true。即在使用一个 topic 时，即使没使用命令创建，也可以继续使用，不会报错。