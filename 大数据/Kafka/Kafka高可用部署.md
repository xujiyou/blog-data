# Kafka 高可用部署

先下载 Kafka 二进制文件：

```bash
$ wget https://mirrors.tuna.tsinghua.edu.cn/apache/kafka/2.5.0/kafka_2.12-2.5.0.tgz
```

解压并重命名：

```bash
$ tar zxvf kafka_2.12-2.5.0.tgz 
$ sudo mv kafka_2.12-2.5.0 /opt/kafka
```

编辑配置文件 `config/server.properties` ：

```
broker.id=0
listeners=PLAINTEXT://fueltank-1:9092
advertised.listeners=PLAINTEXT://fueltank-1:9092
log.dirs=/mnt/vde/kafka
zookeeper.connect=fueltank-1:2181,fueltank-2:2181,fueltank-3:2181
```

按照这个套路再为其他两台修改配置。

创建数据目录：

```bash
$ sudo mkdir -p /mnt/vde/kafka/log
$ sudo chown -R admin:admin /mnt/vde/kafka
```

然后在每台机器上都执行：

```
$ ./bin/kafka-server-start.sh config/server.properties
```

或者后台启动：

```bash
$ ./bin/kafka-server-start.sh -daemon config/server.properties
```



通过 ZooKeeper 查看 Kafka 具有几个点：

```bash
$ ./bin/zkCli.sh
[zk: localhost:2181(CONNECTED) 2] ls /brokers/ids
[0, 1, 2]
```

查看所有 topics：

```bash
[zk: localhost:2181(CONNECTED) 3] ls /brokers/topics
```





## 配置记录

```properties
broker.id=2
listeners=PLAINTEXT://0.0.0.0:9092
advertised.listeners=PLAINTEXT://aitou.push2.bbdservice.net:9092
advertised.host.name=125.65.43.194

broker.id=3
listeners=PLAINTEXT://0.0.0.0:9092
advertised.listeners=PLAINTEXT://aitou.push3.bbdservice.net:9092
advertised.host.name=125.65.43.195
```





