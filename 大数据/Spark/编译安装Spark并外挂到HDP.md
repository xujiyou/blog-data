# 编译安装 Spark 并外挂到 HDP

准备在线上 HDP 大数据集群中升级 Spark 的版本。

HDP 版本：3.1.5

HDP 中 Spark 版本：2.3.2.3.1.5.0-152，Scala 版本 2.11.12

准备升级到当前最新版本 3.0.1



## 编译

下载源码：https://codeload.github.com/apache/spark/tar.gz/v3.0.1

查看 hadoop 版本：

```bash
$ hadoop version
Hadoop 3.1.1.3.1.5.0-152
```



解压之后，进入目录，执行命令：

```bash
$ ./dev/change-scala-version.sh 2.12
$ export MAVEN_OPTS="-Xmx2g -XX:ReservedCodeCacheSize=1g"
$ ./build/mvn -Phadoop-3.1 -Dhadoop.version=3.1.1.3.1.5.0-152 -Phive -Phive-thriftserver -Pkubernetes -Pyarn -DskipTests clean package
```

报错：

```
[WARNING] The requested profile "hadoop-3.1" could not be activated because it does not exist.
[ERROR] Failed to execute goal on project spark-launcher_2.12: Could not resolve dependencies for project org.apache.spark:spark-launcher_2.12:jar:3.0.1: Could not find artifact org.apache.hadoop:hadoop-client:jar:3.1.1.3.1.5.0-152 in gcs-maven-central-mirror (https://maven-central.storage-download.googleapis.com/maven2/) -> [Help 1]
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/DependencyResolutionException
[ERROR] 
[ERROR] After correcting the problems, you can resume the build with the command
[ERROR]   mvn <args> -rf :spark-launcher_2.12
```

解决方案，在 pom.xml 中添加 HDP 公司的仓库：

```xml
   <repository>
     <releases>
      <enabled>true</enabled>
     </releases>
     <snapshots>
      <enabled>true</enabled>
     </snapshots>
     <id>hortonworks.extrepo</id>
     <name>Hortonworks HDP</name>
     <url>http://repo.hortonworks.com/content/repositories/releases</url>
   </repository>
   <repository>
     <releases>
      <enabled>true</enabled>
     </releases>
     <snapshots>
      <enabled>true</enabled>
     </snapshots>
     <id>hortonworks.other</id>
     <name>Hortonworks Other Dependencies</name>
     <url>http://repo.hortonworks.com/content/groups/public</url>
   </repository>
```

编译完成后，进行打包：

```bash
$ ./dev/make-distribution.sh --name hadoop-3.1.1.3.1.5.0-152_spark3.0.1 --tgz -Phadoop-provided -Phadoop-3.1 -Dhadoop.version=3.1.1.3.1.5.0-152 -Phive -Phive-thriftserver -Pyarn -Pkubernetes
```

经过漫长的等待，会生成一个压缩包：spark-3.0.1-bin-hadoop-3.1.1.3.1.5.0-152_spark3.0.1.tgz



## 安装

将这个压缩包拷贝至生产机器并解压到 `/opt/spark-3.0.1` 目录。

修改文件夹所属用户：

```bash
$ sudo chown -R spark:spark /opt/spark-3.0.1
```

根据需求拷贝线上集群对应组件的相关xml文件到解压的。

```
/etc/hadoop/conf/core-site.xml
/etc/hadoop/conf/mapred-site.xml
/etc/hadoop/conf/yarn-site.xml
/etc/hadoop/conf/hdfs-site.xml
/etc/hive/conf/hive-site.xml
/etc/hbase/conf/hbase-site.xml
```

创建 HDFS 文件夹：

```bash
$ hadoop fs -mkdir -p /user/spark3-history
```

修改配置文件：

```bash
$ cp conf/spark-defaults.conf.template conf/spark-defaults.conf
$ vim conf/spark-defaults.conf
```

加入以下内容：

```
spark.eventLog.enabled  true
spark.eventLog.compress true
spark.eventLog.dir      hdfs://hdp1.testing.com:8020/user/spark3-history
spark.history.fs.logDirectory   hdfs://hdp1.testing.com:8020/user/spark3-history
#若是HDP，增加
spark.driver.extraJavaOptions -Dhdp.version=3.1.1.3.1.5.0-152
spark.yarn.am.extraJavaOptions -Dhdp.version=3.1.1.3.1.5.0-152
```

编辑 `spark-env.sh` :

```bash
$ cp conf/spark-env.sh.template conf/spark-env.sh
$ vim conf/spark-env.sh
```

添加以下内容：

```
HADOOP_CONF_DIR=/etc/hadoop/conf/
SPARK_HISTORY_OPTS="-Dspark.yarn.historyServer.address=hdp1.testing.com:18080 -Dspark.history.fs.logDirectory=hdfs://hdp1.testing.com:8020/user/spark3-history"
SPARK_DAEMON_CLASSPATH=$(hadoop classpath)
SPARK_DIST_CLASSPATH=$(hadoop classpath)
```

创建 hdfs 目录：

```bash
$ hadoop dfs -mkdir /user/spark3-history
$ sudo -u hdfs hdfs dfs -chown -R spark:hdfs /user/spark3-history
$ sudo -u hdfs hdfs dfs -chmod -R 775 /user/spark3-history
```



启动 spark history server：

```bash
$ sudo -u hdfs /opt/spark-3.0.1/sbin/start-history-server.sh
```





## 测试

```bash
$ cat >/tmp/test.json<<EOF
{"a":1,"b":2}
EOF

$ cat >/tmp/test.py<<EOF
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("test").getOrCreate()
src_df = spark.read.json("/tmp/test.json").rdd.filter(lambda x: x is not None)
src_df.saveAsTextFile("/tmp/testforspark301")
EOF

$ hadoop fs -put tmp/test.json /tmp/

$ cat >/tmp/run.sh<<EOF
#!/bin/bash
# 指定python环境，避免各个节点调用python 不一致；

export PYSPARK_PYTHON=/bin/python
export PYSPARK_DRIVER_PYTHON=/bin/python
cd /opt/spark-3.0.1
./bin/spark-submit --master yarn /tmp/test.py
EOF
```

> 如果遇到 log4j、slf4j、guave 的类找不到，那就去 Maven 里边手动下载，然后放到 `/opt/spark-3.0.1/jars` 中。

检查：

```
$ hadoop fs -cat /tmp/testforspark301/*
Row(a=1, b=2)
$ hadoop fs -cat /tmp/testforspark301/part-00000
Row(a=1, b=2)
$ hdfs dfs -ls /user/spark3-history
-rwxrwxr-x   3 spark hdfs      55211 2020-09-28 15:38 /user/spark3-history/application_1601177781437_0022.lz4
```

浏览器打开：http://127.0.0.1:18080













