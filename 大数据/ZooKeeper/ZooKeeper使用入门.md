# ZooKeeper 使用入门

ZooKeeper 安装完成之后，运行命令行客户端：

```
$ zookeeper-client -server localhost:2181
```

然后：

```
[zk: localhost:2181(CONNECTED) 0] help
ZooKeeper -server host:port cmd args
	stat path [watch]
	set path data [version]
	ls path [watch]
	delquota [-n|-b] path
	ls2 path [watch]
	setAcl path acl
	setquota -n|-b val path
	history 
	redo cmdno
	printwatches on|off
	delete path [version]
	sync path
	listquota path
	rmr path
	get path [watch]
	create [-s] [-e] path data acl
	addauth scheme auth
	quit 
	getAcl path
	close 
	connect host:port
[zk: localhost:2181(CONNECTED) 1] 
```

help 列出了所有的命令。

查看目录：

```
[zk: localhost:2181(CONNECTED) 1] ls /
[zookeeper, solr, kafka, hive_zookeeper_namespace_hive, ngdata, hbase]
[zk: localhost:2181(CONNECTED) 2] 
```

查看信息：

```
[zk: localhost:2181(CONNECTED) 7] get /kafka
null
cZxid = 0x3570a
ctime = Wed Dec 18 21:39:39 CST 2019
mZxid = 0x3570a
mtime = Wed Dec 18 21:39:39 CST 2019
pZxid = 0x35a72
cversion = 18
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 10
[zk: localhost:2181(CONNECTED) 8] get /hbase

cZxid = 0x14e
ctime = Tue Dec 03 11:16:59 CST 2019
mZxid = 0x14e
mtime = Tue Dec 03 11:16:59 CST 2019
pZxid = 0x35ae5
cversion = 47
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 15
[zk: localhost:2181(CONNECTED) 9] 
```

创建目录：

```
[zk: localhost:2181(CONNECTED) 1] create /test test
Created /test
[zk: localhost:2181(CONNECTED) 2] ls /
[zookeeper, test, solr, kafka, hive_zookeeper_namespace_hive, ngdata, hbase]
```

设置数据：

```
[zk: localhost:2181(CONNECTED) 10] set /test junk=1
cZxid = 0x3bfb0
ctime = Fri Dec 20 17:51:22 CST 2019
mZxid = 0x3bfc7
mtime = Fri Dec 20 17:53:25 CST 2019
pZxid = 0x3bfbc
cversion = 1
dataVersion = 2
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 6
numChildren = 1
```

获取数据：

```
[zk: localhost:2181(CONNECTED) 11] get /test
junk=1
cZxid = 0x3bfb0
ctime = Fri Dec 20 17:51:22 CST 2019
mZxid = 0x3bfc7
mtime = Fri Dec 20 17:53:25 CST 2019
pZxid = 0x3bfbc
cversion = 1
dataVersion = 2
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 6
numChildren = 1
```

删除数据：

```
[zk: localhost:2181(CONNECTED) 14] delete /test
```

查看 Zookeeper 版本：

```bash
$ echo stat|nc localhost 2181
```

