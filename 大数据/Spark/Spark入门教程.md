# Spark 入门教程

Spark 生态系统包括 Shark/Spark SQL，Spark Streaming，GraphX，MLlib，和 Spark Core。

Spark 生态系统完全兼容 Hadoop 生态系统，可以和 HDFS，HBase，YARN 无缝对接。

Spark 对标的是 Hadoop 中的 MapReduce。

需要学习的点包括 RDD 及编程接口，运行模式及原理，调度、储存、监控管理。然后就是命令行工具的运用，和各个子框架的学习和运用。

当然，要想学的通汇贯通还需要学习 Scala ，因为源码是第一手资料。

## Hello World

作为入门，先用本地模式跑一下。

首先用 Maven 建立项目，然后加入下列依赖，依赖要下载好长时间

```xml
<properties>
  <spark.version>2.4.0</spark.version>
  <scala.version>2.11</scala.version>
</properties>

<dependencies>
  <dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-core_${scala.version}</artifactId>
    <version>${spark.version}</version>
  </dependency>
  <dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-streaming_${scala.version}</artifactId>
    <version>${spark.version}</version>
  </dependency>
  <dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-sql_${scala.version}</artifactId>
    <version>${spark.version}</version>
  </dependency>
  <dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-hive_${scala.version}</artifactId>
    <version>${spark.version}</version>
  </dependency>
  <dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-mllib_${scala.version}</artifactId>
    <version>${spark.version}</version>
  </dependency>

  <dependency>
    <groupId>com.thoughtworks.paranamer</groupId>
    <artifactId>paranamer</artifactId>
    <version>2.8</version>
  </dependency>
</dependencies>
```

Spark 版本可以自己选，最好和 CDH 集群中的 Spark 版本一致，Scala 的版本要和运行环境的版本一致，不然会报错。

com.thoughtworks.paranamer 这个包不知道干啥的，反正不加会报错，网上搜的。

然后准备一些数据，这里选几条，有人看见了可以拷贝到本地运行，这是成都的部分小区的房价清单。

```
{"小区": "融创玖樾台邸", "地址": "锦江 - 三圣乡 - 三圣花乡银木街与茶花街交汇处", "建面": "建面 50-63㎡", "单价": "10000", "总价": "总价63万/套"}
{"小区": "中南樾府", "地址": "金牛 - 天回镇 - 金牛区聚霞路933号", "建面": "建面 120-148㎡", "单价": "13800", "总价": "总价155万/套"}
{"小区": "成都文旅城", "地址": "都江堰 - 都江堰 - 成都市都江堰市青外江大桥与青城山大道交汇处(玉堂LG博爱中学旁)", "建面": "建面 75-142㎡", "单价": "9000", "总价": "总价72万/套"}
{"小区": "万科公园都会", "地址": "天府新区 - 麓湖生态城 - 天府新区鹭湖南路东段360-361号", "建面": "建面 105-136㎡", "单价": "21000", "总价": "总价210万/套"}
{"小区": "东湖郡", "地址": "新都 - 新都城区 - 新繁镇文庙街88号", "建面": null, "单价": "19000", "总价": null}
{"小区": "新希望锦悦北府", "地址": "新都 - 新都城区 - 兴贸路1288号", "建面": "建面 117-134㎡", "单价": "12500", "总价": "总价140万/套"}
{"小区": "东城国际(崇州)", "地址": "崇州 - 崇州 - 崇州市崇阳镇江源中路99号", "建面": null, "单价": "15000", "总价": null}
{"小区": "三盛都会城", "地址": "龙泉驿 - 东山 - 龙华路东侧三盛都会城", "建面": null, "单价": "20000", "总价": null}
{"小区": "驿都城", "地址": "龙泉驿 - 龙泉驿城区 - 驿都城中路898号", "建面": null, "单价": "36000", "总价": null}
{"小区": "融创玖阙府一期", "地址": "成华 - 动物园 - 熊猫大道融创玖阙府一期", "建面": null, "单价": "26000", "总价": null}
{"小区": "远大中央公园", "地址": "天府新区 - 南湖 - 南湖大道333号", "建面": "建面 102-195㎡", "单价": "价格待定", "总价": null}
{"小区": "黄龙溪谷", "地址": "天府新区南区 - 彭山 - 剑南大道南延线黄龙大道四段(黄龙古镇旁)", "建面": "建面 195-292㎡", "单价": "价格待定", "总价": null}
{"小区": "交大归谷建设派", "地址": "成华 - 理工大 - 二仙桥北路3号", "建面": null, "单价": "39999", "总价": null}
{"小区": "星宸国际", "地址": "高新 - 新会展 - 益州大道中段555号", "建面": null, "单价": "15000", "总价": null}
{"小区": "城北优品道", "地址": "新都 - 大丰 - 北星大道与大天路交汇处(十字路口出城方向右侧）", "建面": null, "单价": "20000", "总价": null}
{"小区": "世豪金河谷", "地址": "温江 - 光华大道沿线 - 涌泉街道江浦路2666号", "建面": "建面 357-411㎡", "单价": "700", "总价": "总价839.02万/套"}
{"小区": "佳年华新生活", "地址": "温江 - 花都大道 - 江安路518号", "建面": null, "单价": "20000", "总价": null}
{"小区": "银泰中心", "地址": "高新 - 金融城 - 天府大道北段1199号（高新国际广场旁）", "建面": "建面 580-660㎡", "单价": "70000", "总价": "总价4000万/套"}
{"小区": "银泰中心", "地址": "高新 - 金融城 - 天府大道北段1199号（高新国际广场旁）", "建面": "建面 76-251㎡", "单价": "47000", "总价": "总价1260万/套"}
{"小区": "中德麓府", "地址": "天府新区 - 麓山 - 麓山大道二段（新元素奥迪旗舰店旁）", "建面": null, "单价": "价格待定", "总价": null}
```

然后编写 Scala 代码：

```scala
package work.xujiyou.spark

import org.apache.spark.{SparkConf, SparkContext}

import scala.util.parsing.json.JSON

object HomeSort {

  def strToInt(str: String): Int = {
    val regex = """([0-9]+)""".r
    val res = str match {
      case regex(num) => num
      case _ => "0"
    }
    val resInt = Integer.parseInt(res)
    resInt
  }

  def main(args: Array[String]) {
    val inputFile = "/Users/jiyouxu/PycharmProjects/spider/home_list.txt"
    val conf = new SparkConf().setAppName("HomeSort").setMaster("local[2]")
    val sc = new SparkContext(conf)
    val textFile = sc.textFile(inputFile)
    val homeSortList = textFile
      .map(line => JSON.parseFull(line).get.asInstanceOf[Map[String, String]])
      .sortBy(map => strToInt(map("单价")))
    homeSortList.foreach(println)
    println(homeSortList.count())
    println(textFile.partitions.length)
  }
}
```

好了，这样就可以了，程序的作用是对房价进行排序。

每个 Spark 程序都要初始化一个 SparkContext 才能使用，Master 是 Spark 的运行模式，这里是 Local ，至于后面的 2 ，则是 Spark 用到的线程数，每种运行模式都有一些参数，后面再学习 Spark 的运行模式。

还有其他 master ，可见：https://spark.apache.org/docs/latest/submitting-applications.html#master-urls

## 在 Hadoop 集群中运行 Spark 程序

首先，数据是放在 HDFS 中的，Spark 会自动识别 HDFS 的路径，修改代码如下：

```scala
package work.xujiyou.spark

import org.apache.spark.{SparkConf, SparkContext}

import scala.util.parsing.json.JSON

object HomeSort extends Serializable {

  def strToInt(str: String): Int = {
    val regex = """([0-9]+)""".r
    val res = str match {
      case regex(num) => num
      case _ => "0"
    }
    val resInt = Integer.parseInt(res)
    resInt
  }

  def main(args: Array[String]) {
    val inputFile = "hdfs://fueltank-2:8020/user/root/spider/home_list.txt"
    val conf = new SparkConf().setAppName("HomeSort")
    val sc = new SparkContext(conf)
    val textFile = sc.textFile(inputFile)
    val homeSortList = textFile
      .map(line => JSON.parseFull(line).get.asInstanceOf[Map[String, String]])
      .sortBy(map => strToInt(map("单价")))
    homeSortList.foreach(println)
    homeSortList.saveAsTextFile("hdfs://fueltank-2:8020/user/root/spider/home_sort")
  }
}

```

然后运行 maven package 进行打包，将打好的 jar 包上传到 CDH 集群所在的服务器。

这里一定要注意 Maven 中的 Spark 版本和 Scala 版本和 CDH 集群中的版本一致！

还要注意类要继承 `Serializable `

然后运行 shell 命令：

```bash
$ sudo spark-submit --class work.xujiyou.spark.HomeSort --master yarn --deploy-mode cluster spark-1.0-SNAPSHOT.jar
```

Shell 命令行是不会打印 Spark 程序的日志的，需要将结果日志输出到 HDFS 来查看效果。

如果想看日志，可以使用以下命令：

```bash
$ sudo yarn logs -applicationId <app ID>
```

