# Kafka 主题

创建主题：

```bash
$ kafka-topics.sh --zookeeper fueltank-1:2181 --create --topic one --replication-factor 2 --partitions 2
```

这里的五个参数是必须要指定的，`--zookeeper` 选项在任何时候都需要指定。`--create` 表示创建主题，`--topic` 指定主题名称，`--replication-factor` 指定主题的副本数量，`--partitions` 指定主题的分区数量。

集群会为每个分区创建指定数量的副本。这里一共就是 4 个副本。



## 分区

分区是个什么鬼那。

Kafka 中可以将 Topic 从物理上划分成一个或多个分区（Partition），每个分区在物理上对应一个文件夹，以 "topicName-partitionIndex" 的命名方式命名，该文件夹下存储这个分区的所有消息(.log)和索引文件(.index)，这使得Kafka的吞吐率可以水平扩展。

比如我这里，在 fueltank-1 的 kafka 日志目录中有 one-0，在 fueltank-2 上有 one-0 和 one-1，在 fueltank-3 上有 one-1。

生产者在生产数据的时候，可以为每条消息指定 Key ，这样消息被发送到 broker 时，会根据分区规则选择被存储到哪一个分区中，如果分区规则设置的合理，那么所有的消息将会被均匀的分布到不同的分区中，这样就实现了负载均衡和水平扩展。另外，在消费者端，同一个消费组可以多线程并发的从多个分区中同时消费数据。



## 增加分区

一点需要注意，为Topic创建分区时，分区数最好是broker数量的整数倍，这样才能是一个Topic的分区均匀的分布在整个Kafka集群中。

通过以下命令重新设置分区。

```bash
$ kafka-topics.sh --zookeeper fueltank-1:2181 --alter --topic one --partitions 3
```





## 列出所有主题

```bash
$ kafka-topics.sh --zookeeper fueltank-1:2181 --list
```

查看主题的详细信息：

```bash
$ kafka-topics.sh --zookeeper fueltank-1:2181 --describe
```

```bash
$ kafka-topics.sh --zookeeper localhost:2181 --topic forum_topic --describe
```





## 删除主题









