# Kafka 消费者

需要先学习下关于 Kafka 消费者的一些概念。

## 消费者群组

多个生产者可以往同一个主题里发送消息，也可以有多个消费者读取同一个主题里的消息。

往消费者群组里添加消费者是横向伸缩消费能力的主要方式。

也可以多个消费者群组消费同一个主题，同一个topic，每个消费者组都可以拿到相同的全部数据。

下面来做测试

#### 消费者多于分区数

创建主题，分区和复制数都为 1:

```bash
$ kafka-topics.sh --zookeeper drift-1:2181,drift-2:2181,drift-3:2181 --create --topic my-test --replication-factor 1 --partitions 1
```

修改 consumer.properties 文件：

```
group.id=group1
```

创建两个消费者：

```bash
$ kafka-console-consumer.sh --bootstrap-server drift-1:9092,drift-2:9092,drfit-3:9092 --topic my-test --consumer.config config/consumer.properties
$ kafka-console-consumer.sh --bootstrap-server drift-1:9092,drift-2:9092,drfit-3:9092 --topic my-test --consumer.config config/consumer.properties
```

创建生产者：

```
$ kafka-console-producer.sh --bootstrap-server drift-1:9092,drift-2:9092,drfit-3:9092 --topic test
```

此时，消费者的数量大于分区的数量，使用同一个消费者组，发现只有一个消费者能获得数据。

这说明：同一个分区内的消息只能被同一个组中的一个消费者消费，当消费者数量多于分区数量时，多于的消费者空闲（不能消费数据）。



#### 消费者少于和等于分区数

创建一个三分区的主题：

```
$ kafka-topics.sh --zookeeper drift-1:2181,drift-2:2181,drift-3:2181 --create --topic my-test2 --replication-factor 3 --partitions 3
```

还是在 group1 中启动两个消费者：

```
$ kafka-console-consumer.sh --bootstrap-server drift-1:9092,drift-2:9092,drfit-3:9092 --topic my-test2 --consumer.config config/consumer.properties
$ kafka-console-consumer.sh --bootstrap-server drift-1:9092,drift-2:9092,drfit-3:9092 --topic my-test2 --consumer.config config/consumer.properties
```

经测试发现，当分区数多于消费者数的时候，有的消费者对应多个分区。

如果是在一个群组中，当分区数等于消费者数的时候，每个消费者对应一个分区。



#### 多个消费者组

当多个消费者各自处于不同的消费者时，每个消费者组都会获得全部的数据。













