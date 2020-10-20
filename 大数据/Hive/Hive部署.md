# Hive 部署

首先下载：

```bash
$ wget https://mirror.bit.edu.cn/apache/hive/hive-3.1.2/apache-hive-3.1.2-bin.tar.gz
```

解压并重命名：

```bash
$ tar zxvf apache-hive-3.1.2-bin.tar.gz
$ mv apache-hive-3.1.2-bin hive
```

进入 hive 目录



## 使用 MySQL 储存 Hive 元数据

首先准备一个 MySQL 库，可以参考： [MySQL最新版本安装.md](../../数据存储/MySQL/MySQL最新版本安装.md) 

为 Hive 创建一个用户：

````mysql
mysql> CREATE USER 'hive' IDENTIFIED BY 'BBDERS1@bbdops.com';
````

创建数据库：

```mysql
mysql> create database hive;
```

在 MySQL 8 中，这样授权：

```mysql
mysql> GRANT ALL ON hive.* TO hive@'%';
mysql> FLUSH PRIVILEGES;
```

测试链接：

```bash
$ mysql -uhive -pBBDERS1@bbdops.com --host fueltank-3
```

能正常进去，说明配置成功



## 安装 Hive

添加环境变量：

```bash
sudo vim /etc/profile

#添加
export HIVE_HOME=/opt/hive
export PATH=$PATH:$HIVE_HOME/bin

#生效
source /etc/profile
```

创建数据目录：

```bash
$ sudo mkdir /mnt/vde/hive
$ sudo chown -R admin:admin /mnt/vde/hive
```

配置 Hive：

```bash
$ cp conf/hive-env.sh.template conf/hive-env.sh
```

将以下配置写入 `conf/hive-env.sh` ：

```bash
$ export HADOOP_HOME=/opt/hadoop
```

然后另一个：

```bash
$ cp conf/hive-default.xml.template conf/hive-site.xml
```

修改其中的配置如下：

```xml
  <property>
      <name>javax.jdo.option.ConnectionURL</name>
      <value>jdbc:mysql://fueltank-3:3306/hive</value>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.jdbc.Driver</value> <!--MySQL 8.0 应该为：com.mysql.cj.jdbc.Driver -->
  </property>
  <property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>password</value>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>hive</value>
  </property>
  <property>
    <name>hive.exec.local.scratchdir</name>
    <value>/mnt/vde/hive</value>
  </property>
  <property>
    <name>hive.downloaded.resources.dir</name>
    <value>/mnt/vde/hive</value>
  </property>
  <property>
    <name>hive.exec.scratchdir</name>
    <value>hdfs://fueltank-1:9000/data/hive/temp</value>
  </property>
  <property>
    <name>hive.metastore.warehouse.dir</name>
    <value>hdfs://fueltank-1:9000/data/hive/warehouse</value>
  </property>
  <property>
    <name>hive.querylog.location</name>
    <value>hdfs://fueltank-1:9000/data/hive/log</value>
  </property>
```

日志配置：

```bash
$ mv conf/hive-log4j2.properties.template conf/hive-log4j2.properties
$ mv conf/hive-exec-log4j2.properties.template conf/hive-exec-log4j2.properties
```

修改其中的配置：

```bash
[admin@fueltank-1 conf]$ vim hive-log4j2.properties
property.hive.log.dir =/home/hadoop/hive/log
[admin@fueltank-1 conf]$ vim hive-exec-log4j2.properties
property.hive.log.dir = /home/hadoop/hive/log/exec
```

我使用是 MySQL 8.0，需要下载一个 mysql-connector-java-8.0.20.jar：

```bash
$ wget https://repo1.maven.org/maven2/mysql/mysql-connector-java/8.0.20/mysql-connector-java-8.0.20.jar
```

然后丢到 hadoop 相应的目录中：

```bash
$ mv mysql-connector-java-8.0.20.jar /opt/hadoop/share/hadoop/yarn/lib/
```



## 启动 Hive

先初始化元数据库：

```bash
$ schematool -dbType mysql -initSchema
```

但是出现错误：

```
Exception in thread "main" java.lang.NoSuchMethodError: com.google.common.base.Preconditions.checkArgument(ZLjava/lang/String;Ljava/lang/Object;)V
```

关键在： com.google.common.base.Preconditions.checkArgument 这是因为hive内依赖的**guava.jar**和hadoop内的**版本不一致**造成的。 检验方法：

1. 查看hadoop安装目录下share/hadoop/common/lib内guava.jar版本
2. 查看hive安装目录下lib内guava.jar的版本 如果两者不一致，删除版本低的，并拷贝高版本的 问题解决！

```bash
$ cp /opt/hadoop/share/hadoop/common/lib/guava-27.0-jre.jar lib/
$ mv lib/guava-19.0.jar lib/guava-19.0.jar.backup
```

又报错：

```
Exception in thread "main" java.lang.RuntimeException: com.ctc.wstx.exc.WstxParsingException: Illegal character entity: expansion character (code 0x8
 at [row,col,system-id]: [3215,96,"file:/opt/hive/conf/hive-site.xml"]
```

解决方案：删掉这一行，或者删掉这个字符

![image-20200518140739765](/Users/jiyouxu/Library/Application Support/typora-user-images/image-20200518140739765.png)

然后再运行：

```bash
$ schematool -dbType mysql -initSchema
```

会初始化很长时间，完成后，会有以下日志：

```
Initialization script completed
schemaTool completed
```

一共在 MySQL 中创建了 74 张表。

待数据库初始化完毕后，启动 Hive：

```bash
$ hive
```

这里直接启动一个客户端即可，这条命令也相当于：

```bash
$ hive --service cli
```





## 写入数据

```sql
hive> create database one;
hive> show databases;
hive> use one;
hive> Create Table one_table(one int, two int);
hive> Insert into table one_table values (1,2);
```

插入不成功，报错：

```
FAILED: Execution Error, return code 2 from org.apache.hadoop.hive.ql.exec.mr.MapRedTask
```

需要查看 yarn 界面查找错误：http://fueltank-2:8088/cluster

根据错误提示找出解决方法，修改 Hadoop 的配置文件 `etc/hadoop/mapred-site.xml` ，添加以下配置：

```xml
    <property>
        <name>yarn.app.mapreduce.am.env</name>
        <value>HADOOP_MAPRED_HOME=/opt/hadoop</value>
    </property>
    <property>
        <name>mapreduce.map.env</name>
        <value>HADOOP_MAPRED_HOME=/opt/hadoop</value>
    </property>
    <property>
        <name>mapreduce.reduce.env</name>
        <value>HADOOP_MAPRED_HOME=/opt/hadoop</value>
    </property>
```

修改完成后，不必重启，重新插入即可。

插入完成后，数据放在 HDFS 的 /data/hive/warehouse/one.db/one_table/000000_0 中了，查看：

````bash
$ hdfs dfs -cat /data/hive/warehouse/one.db/one_table/000000_0
````

也可以在 Hive 中查看：

```sql
hive> use one;
hive> select * from one_table;
```







