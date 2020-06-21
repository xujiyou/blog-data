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

在把 kafka 部署到 k8s 后，需要在外部访问 k8s 的kafka，就用到了这个配置：

![image-20200618165139483](../../../resource/image-20200618165139483.png)



#### auto.create.topics.enable

在服务器上开启自动创建 topic，默认为 true。即在使用一个 topic 时，即使没使用命令创建，也可以继续使用，不会报错。



#### auto.leader.rebalance.enable

如果开启了，后台线程定期检查分区领导者的分布，如果超过了`leader.imbalance.per.broker.percentage`设定的失衡比例， 则会导致领导者重新平衡到分区的首选领导者。



#### background.threads

用于各种后台处理任务的线程数



#### broker.id

默认为 -1，如果没有设置，则会生成一个唯一 ID，为避免Zookeeper生成的代理ID与用户配置的代理ID之间发生冲突，生成的代理ID从reserved.broker.max.id + 1开始。



#### compression.type

指定给定 topic 的的压缩类型。此配置接受标准压缩编解码器（“ gzip”，“ snappy”，“ lz4”，“ zstd”）。它还接受“uncompressed”，这相当于不压缩。'producer'，表示保留生产者设置的原始压缩编解码器。











