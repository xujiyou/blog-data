---
title: MirrorMaker实战
date: 2020-06-17 19:55:26
tags: Kafka
---

 MirrorMaker是Kafka附带的一个用于在Kafka集群之间制作镜像数据的工具。该工具从源集群中消费并生产到目标群集。这种镜像的常见用例是在另一个数据中心提供副本。

使用方式很简单：

```bash
$ kafka-mirror-maker.sh --consumer.config config/consumer.properties --producer.config config/producer.properties --whitelist "one"
```

`consumer.properties` 和 `producer.properties` 都是 kafka 包内自带的，可根据注释进行修改。

或者使用 ：

```bash
$ connect-mirror-maker.sh config/connect-mirror-maker.properties
```

`connect-mirror-maker.properties` 也是 kafka 自带的一个配置文件，可以修改使用。

