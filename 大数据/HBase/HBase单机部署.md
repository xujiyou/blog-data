# HBase 单机部署

单机部署用于测试/开发环境。

HBase 是支持使用本地文件系统路径的，当然正式环境里还是要依赖 HDFS。

需要注意的坑是：HBase 自带的 Zookeeper 不允许外部访问，所以还是用外部的 Zookeeper，更可控一些。

在 hbase-site.xml 中添加：

```xml
<property>
    <name>hbase.rootdir</name>
    <value>file:///data2/hbase</value>
  </property>
  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>
  <property>
    <name>hbase.tmp.dir</name>
    <value>/data2/hbase-tmp</value>
  </property>
  <property>
    <name>hbase.unsafe.stream.capability.enforce</name>
    <value>false</value>
  </property>
    <property>
        <name>hbase.zookeeper.quorum</name>
        <value>10.28.109.27:2181</value>
    </property>
    <property>
        <name>hbase.zookeeper.property.dataDir</name>
        <value>/data2/zookeeper</value>
    </property>
```

在 hbase-env.sh 中添加：

```
export JAVA_HOME=/usr/java/jdk1.8.0_261-amd64
export HBASE_MANAGES_ZK=false
```

