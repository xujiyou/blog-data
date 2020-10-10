# NacOS 部署

- Nacos 版本 1.1.0
- 服务器：192.168.6.121
- 安装位置：/opt/s1/nacos

去 Github 上下载 Nacos 的二进制包。

在 `192.168.6.128` 上创建数据库：

```
$ docker exec -i -t 54542b63ff3a mysql -uroot -p
mysql> create database nacos_devtest default character set utf8mb4;
mysql> grant all on nacos_devtest.* to 'nacos_devtest'@'%' identified by 'slcrawler';
mysql> grant all on nacos_devtest.* to 'nacos_devtest'@'localhost' identified by 'slcrawler';
```

执行 sql 文件：

```
$ scp nacos-mysql.sql schema.sql root@192.168.6.128:/root
$ docker cp schema.sql  54542b63ff3a7733b35def992519c55ec4ee4060ff6a93517ddd15d96eebc618:/tmp/
$ docker cp nacos-mysql.sql  54542b63ff3a7733b35def992519c55ec4ee4060ff6a93517ddd15d96eebc618:/tmp/
$ docker exec -i -t 54542b63ff3a mysql -unacos_devtest -p
mysql> use nacos_devtest;
mysql> source /tmp/nacos-mysql.sql
mysql> source /tmp/schema.sql
```

重命名配置文件 ：

```bash
$ cd config
$ mv cluster.conf.example cluster.conf
```

修改 cluster.conf 的内容为 :

```
192.168.6.121
```

修改 `application.properties` 文件，添加以下内容：

```
db.num=1
db.url.0=jdbc:mysql://192.168.6.128:3306/nacos_devtest?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=nacos_devtest
db.password=slcrawler
```

启动：

```bash
$ bash startup.sh
```

启动日志：`/opt/s1/nacos/logs/start.out`

验证：

```bash
$ tail -f /opt/s1/nacos/logs/start.out
$ lsof -i:8848
$ ps -ef | grep nacos
```

