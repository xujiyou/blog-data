---
title: Flink 高可用部署
date: 2019-11-27 12:13:57
categories: 大数据
---

# Flink 高可用部署

先下载二进制包：

```bash
$ wget http://ftp.cuhk.edu.hk/pub/packages/apache.org/flink/flink-1.10.1/flink-1.10.1-bin-scala_2.12.tgz
```



## 基于 Yarn 模式的高可用部署

下面来实践基于 Yarn 的高可用模式。

修改 `conf/flink-conf.yaml` 的配置：

````
high-availability: zookeeper
high-availability.storageDir: hdfs://fueltank-1:9000/flink/ha/
high-availability.zookeeper.quorum: fueltank-1:2181,fueltank-2:2181,fueltank-3:2181
````

往 `/etc/profile` 尾部加入以下代码：

```bash
export HADOOP_CONF_DIR=${HADOOP_HOME}/etc/hadoop
export HADOOP_CLASSPATH=`${HADOOP_HOME}/bin/hadoop classpath`
```

使环境变量生效：

```bash
$ source /etc/profile
```

启动：

````bash
$ ./bin/yarn-session.sh -d
````

-d 的意思是让 flink 运行在 detached 模式下，即脱离 shell session 运行。

在启动日志里有这么一条：

```
JobManager Web Interface: http://fueltank-2.cloud.bbdops.com:33142
```

说明图形界面是 http://fueltank-2.cloud.bbdops.com:33142

停止：

```bash
$ echo "stop" | ./bin/yarn-session.sh -id application_1589788890577_0004
```

或者：

```
$ yarn application -kill application_1589788890577_0004
```



通过上面的实际操作可以看出，上面的方式实际是把 Flink 当作一个 Yarn 任务在运行的。





## 独立模式的高可用部署

停掉上边的任务。

然后修改 `conf/masters`

```
fueltank-1:9081
fueltank-2:9081
fueltank-3:9081
```

这三个地址都是可以看到 web 界面的。

masters 指定了有哪些 **jobmanager**

修改 `conf/slaves`

```
fueltank-2
fueltank-3
```

slaves 指定有哪些 **taskmanager**



启动集群（只需执行一次，需要在每台机器的相同位置安装 Flink）：

```bash
$ ./bin/start-cluster.sh
```

停止集群：

```bash
$ ./bin/stop-cluster.sh 
```

启动集群后，可以启动多个 TaskManager：

```bash
$ ./bin/taskmanager.sh start
```

如果停掉了 JobManager：

```bash
$ ./bin/jobmanager.sh stop
```

可用以下命令重启：

```bash
$ ./bin/jobmanager.sh start fueltank-1 9081
```











