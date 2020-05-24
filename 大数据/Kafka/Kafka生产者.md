# Kafka 生产者

首先在命令行里面生产数据：

```bash
$ kafka-console-producer.sh --bootstrap-server fueltank-1:9092,fueltank-2:9092,fueltank-3:9092 --topic one
```

`--bootstrap-server` 可以指定一个或多个 Kafka broker 的地址，提供两个及以上，可以保证高可用。`--topic` 指定主题。

创建一个消费者：

```bash
$ kafka-console-consumer.sh --bootstrap-server fueltank-1:9092,fueltank-2:9092,fueltank-3:9092 --topic one --from-beginning
```

`--from-beginning` 表示从头开始获取。

从指定的分区中获取数据：

```bash
$ kafka-console-consumer.sh --bootstrap-server fueltank-1:9092,fueltank-2:9092,fueltank-3:9092 --topic one --partition 0
```





## Java 代码

使用 Java 来创建一个生产者。

创建 Maven 项目，加入以下依赖：

```xml
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>2.5.0</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.5</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-simple</artifactId>
            <version>1.6.4</version>
        </dependency>
```

然后 Java 代码：

```java
package com.bbd.kafka;

import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;

import java.util.Properties;
import java.util.concurrent.ExecutionException;

public class Producer {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        Properties kafkaProps = new Properties();
        kafkaProps.put("bootstrap.servers", "fueltank-1:9092,fueltank-2:9092,fueltank-3:9092");
        kafkaProps.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        kafkaProps.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        KafkaProducer<String, String> producer = new KafkaProducer<>(kafkaProps);

        ProducerRecord<String, String> record = new ProducerRecord<>("one", "1", "hello world");
        producer.send(record).get();
    }
}
```

这里 ProducerRecord 的第二个参数是 key，根据这个 key 会发往不同的分区。

## 异步发送

```java
package com.bbd.kafka;

import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;

import java.util.Properties;
import java.util.concurrent.ExecutionException;

public class Producer {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        Properties kafkaProps = new Properties();
        kafkaProps.put("bootstrap.servers", "fueltank-1:9092,fueltank-2:9092,fueltank-3:9092");
        kafkaProps.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        kafkaProps.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        KafkaProducer<String, String> producer = new KafkaProducer<>(kafkaProps);

        ProducerRecord<String, String> record = new ProducerRecord<>("one", "Precision Products", "hello world");
        producer.send(record, (recordMetadata, e) -> {
            System.out.println(recordMetadata);
        }).get();
    }
}
```





