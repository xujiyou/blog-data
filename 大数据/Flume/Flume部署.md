# Flume 部署

Flume 二进制包下载地址：https://flume.apache.org/download.html

解压：

```bash
$ tar zxvf apache-flume-1.9.0-bin.tar.gz
$ sudo mv apache-flume-1.9.0-bin/ /opt/
```

修改配置文件：

```bash
$ sudo mv /opt/apache-flume-1.9.0-bin/conf/flume-env.sh.template /opt/apache-flume-1.9.0-bin/conf/flume-env.sh
$ sudo vim /opt/apache-flume-1.9.0-bin/conf/flume-env.sh
export JAVA_HOME=/usr/java/jdk1.8.0_261-amd64
```

这个东西没有后台进程。

使用方式见：https://flume.apache.org/releases/content/1.9.0/FlumeUserGuide.html

新建一个名为 `example.conf` 的文件：

```properties
# example.conf: A single-node Flume configuration

# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 44444

# Describe the sink
a1.sinks.k1.type = logger

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

然后执行命令：

```bash
$ bin/flume-ng agent --conf conf --conf-file example.conf --name a1 -Dflume.root.logger=INFO,console
```

在另外一个命令行窗口进行测试：

```
$ telnet localhost 44444
Trying ::1...
telnet: connect to address ::1: Connection refused
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
hello world
OK
```

