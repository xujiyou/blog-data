# Hive 错误处理

当我执行 `desc xujiyou;`时，出现错误：

```
FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. java.lang.ClassNotFoundException Class org.apache.hive.hcatalog.data.JsonSerDe not found
```

这时退出 hive ，然后在 shell 里执行：

```bash
$ locate *hive-hcatalog-core*.jar
```

在系统中查找出对应的jar，若没有，去网上下载一个。我的系统里面有：

```
/opt/cloudera/cm/cloudera-navigator-server/libs/cdh5/hive-hcatalog-core-1.1.0-cdh5.12.0.jar
/opt/cloudera/cm/cloudera-navigator-server/libs/cdh6/hive-hcatalog-core-2.1.1-cdh6.1.1.jar
/opt/cloudera/parcels/CDH-6.1.1-1.cdh6.1.1.p0.875250/jars/hive-hcatalog-core-2.1.1-cdh6.1.1.jar
/opt/cloudera/parcels/CDH-6.1.1-1.cdh6.1.1.p0.875250/lib/hbase-solr/lib/hive-hcatalog-core.jar
/opt/cloudera/parcels/CDH-6.1.1-1.cdh6.1.1.p0.875250/lib/hive-hcatalog/share/hcatalog/hive-hcatalog-core-2.1.1-cdh6.1.1.jar
/opt/cloudera/parcels/CDH-6.1.1-1.cdh6.1.1.p0.875250/lib/hive-hcatalog/share/hcatalog/hive-hcatalog-core.jar
/opt/cloudera/parcels/CDH-6.1.1-1.cdh6.1.1.p0.875250/lib/impala/lib/hive-hcatalog-core.jar
/opt/cloudera/parcels/CDH-6.1.1-1.cdh6.1.1.p0.875250/lib/oozie/embedded-oozie-server/webapp/WEB-INF/lib/hive-hcatalog-core-2.1.1-cdh6.1.1.jar
/opt/cloudera/parcels/CDH-6.1.1-1.cdh6.1.1.p0.875250/lib/oozie/lib/hive-hcatalog-core.jar
/opt/cloudera/parcels/CDH-6.1.1-1.cdh6.1.1.p0.875250/lib/oozie/oozie-sharelib-yarn/lib/hcatalog/hive-hcatalog-core.jar
/opt/cloudera/parcels/CDH-6.1.1-1.cdh6.1.1.p0.875250/lib/oozie/oozie-sharelib-yarn/lib/pig/hive-hcatalog-core.jar
/opt/cloudera/parcels/CDH-6.1.1-1.cdh6.1.1.p0.875250/lib/oozie/oozie-sharelib-yarn/lib/sqoop/hive-hcatalog-core.jar
/opt/cloudera/parcels/CDH-6.1.1-1.cdh6.1.1.p0.875250/lib/sentry/lib/hive-hcatalog-core.jar
```

随便选一个新版本的，然后进入hive后执行：

```
ADD JAR /opt/cloudera/cm/cloudera-navigator-server/libs/cdh6/hive-hcatalog-core-2.1.1-cdh6.1.1.jar;
```

然后再运行 `desc xujiyou;` 完美！