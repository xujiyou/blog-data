# ZooKeeper 之 ACL 认证

ZooKeeper的Client-Server互认证机制是从3.4.0版本开始引入的。

ZooKeeper的ACL可针对`znodes`设置相应的权限信息。ACL数据的表示格式为：`schema:id:permissions`

**schema**   

支持的几种 `schema` 为：

- **world** 默认方式，只有一个名为`anyone`的`Id`, `world:anyone`代表任何人，也就是说，对应节点任何人可访问

- **auth **代表任何通过认证的用户，该**schema**不需要配置`Id`信息

- **digest** 基于`username:password`生成的MD5 Hash值作为`Id`信息，认证基于`username:password`明文认证，但在acl中存储的是`username:base64(password)`

- **ip** 基于IP地址作为`Id`，支持IP地址或IP地址段



**id**  代表用户

**permissions**  权限定义为**(READ, WRITE, CREATE, DELETE, ADMIN, ALL)**，这5种权限简写为 crwda (即：每个单词的首字符缩写)

由ACL的定义信息，可以看出来，ZooKeeper可以针对不同的`znodes`来提供不同的认证机制。

查看默认的 ACL：

```
[zk: 172.20.20.145:2181(CONNECTED) 3] getAcl /test
'world,'anyone
: cdrwa
```

可以看到所有人都可以访问，权限为全部权限。

进入 zookeeper 的 libs 目录，执行以下命令来生成密码：

````bash
$ java -cp "./zookeeper-3.6.1.jar:./slf4j-api-1.7.25.jar" org.apache.zookeeper.server.auth.DigestAuthenticationProvider xujiyou:123456
````

下面开始设置 ACL：

```
[zk: 172.20.20.145:2181(CONNECTED) 4] setAcl /test digest:xujiyou:Q8GM2p1xjb6R313Bk5eb0RGWJkg=:crwda
```

格式是 setAcl /test digest:用户名:密码:权限 

这时候再查看 ACL，会发现以下问题：

```
[zk: 172.20.20.145:2181(CONNECTED) 5] getAcl /test
Authentication is not valid : /test
```

需要给当前上下文添加认证：

```
[zk: 172.20.20.145:2181(CONNECTED) 6] addauth digest xujiyou:123456
```

这样就认证通过了！！！









