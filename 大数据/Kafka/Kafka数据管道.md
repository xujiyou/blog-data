# Kafka 数据管道

Kafka 数据管道，即 Kafka Connect。

数据管道带来的主要价值在于，它可以作为数据管道的各个数据段之间的大型缓冲区！有效的解藕管道数据的生产者和消费者。

Kafka 的解藕能力以及在安全和效率方面的可靠性，使它成为数据管道的最佳选择。



## 示例

写入一些数据：

```bash
$ echo -e "foo\nbar" > test.txt
```

启动连接器

```bash
$ connect-standalone.sh config/connect-standalone.properties config/connect-file-source.properties config/connect-file-sink.properties
```

完成后，会在当前目录生成一个 `test.sink.txt` 文件，和原始文件一模一样的内容。

上边这个是 standalone 模式的例子，下面看一个 distributed 模式的例子。

```bash
$ connect-distributed.sh config/connect-distributed.properties
```

