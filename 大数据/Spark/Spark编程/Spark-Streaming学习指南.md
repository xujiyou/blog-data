# Spark Streaming学习指南

Spark Streaming 是核心 Spark API 的扩展，具有可扩展，高吞吐量，容错的特点，数据可以从 Kafka 、Flume、Kinesis、或 TCP 套接字中获取，获取之后可以用高阶函数实现的算法来处理。最后，处理后的数据可以放在文件系统、数据集，或展示给用户。甚至也可以运行 Spark 平台上的机器学习和图算法。

Spark Streaming 提供了高阶抽象叫做 *discretized stream* or *DStream* ，它表示连续的数据流，由来自 Kafka 等源的数据流创建，在 Spark 内部，DStream 就是一个序列化的 RDD。

## 实例程序

在 Spark 入门教程中已经介绍了如何编写 Spark 程序，这里直接写代码：

```scala
package work.xujiyou.spark

import org.apache.spark.SparkConf
import org.apache.spark.streaming.{Seconds, StreamingContext}

object NetworkWordCount extends App {

  val conf = new SparkConf().setMaster("local[2]").setAppName("NetworkWordCount")
  val ssc = new StreamingContext(conf, Seconds(10))
  val lines = ssc.socketTextStream("localhost", 9999)
  val words = lines.flatMap(_.split(" "))
  val pairs = words.map(word => (word, 1))
  val wordCounts = pairs.reduceByKey(_ + _)
  wordCounts.print()
  ssc.start()             // Start the computation
  ssc.awaitTermination()
}
```

运行代码前，需要在命令行运行以下程序：

```bash
$ nc -lk 9999
```

这条命令打开了一个 TCP 套接字，用于给 Spark 程序传递数据

然后在 IDEA 中运行上面的 scala 程序，再给 Spark 传递数据，最后在 IDEA 的运行窗口就看到数据输出了。