# 证监会 CDH 部署

版本 5.15.2

使用 mariadb 5.5

JDK HotSpot 1.8.0_251

JAVA_HOME 配置在 /etc/profile ，地址为 /usr/java/jdk1.8.0_251-amd64

在 bbd01 上启动有 httpd 服务。



## 数据库

mariadb 安装在 bbd01 中。

修改 mariadb 密码：

```mysql
use mysql;
update user set password=password("BBDERS1@bbdops.com")where user='root';
flush privileges;
```

登录使用：

```bash
$ mysql --host bbd01 -uroot -p
```

创建用户及数据库：

```mysql
# 创建用于cloudera运行的mysql数据库cmf
mysql> create database cmf  default character set utf8;
mysql> grant all on cmf.* to 'cmf'@'bbd01' identified by 'CMF1@password';
mysql> grant all on cmf.* to 'cmf'@'localhost' identified by 'CMF1@password';

# 创建其他组件的mysql数据库
mysql> create database hive  default character set utf8;
mysql> grant all on hive.* to 'hive'@'bbd01' identified by 'HIVE1@password';
mysql> grant all on hive.* to 'hive'@'localhost' identified by 'HIVE1@password';

mysql> create database oozie  default character set utf8;
mysql> mysql> grant all on oozie.* to 'oozie'@'bbd01' identified by 'OOZIE1@password';
mysql> grant all on oozie.* to 'oozie'@'localhost' identified by 'OOZIE1@password';

mysql> create database hue  default character set utf8;
mysql> grant all on hue.* to 'hue'@'bbd01' identified by 'HUE1@password';
mysql> grant all on hue.* to 'hue'@'localhost' identified by 'HUE1@password';
```





## httpd

在 bbd01 上启动了一个 httpd 服务，修改配置文件 `/etc/httpd/conf/httpd.conf` :

```
DocumentRoot "/data/http"

<Directory "/data/http">
```

离线安装包储存在 /data/http 中，占用 80 端口。



## CM

cm repo 文件在 /etc/yum.repos.d/cm.repo

cloudera-manager-server 安装在 bbd01 中。

```bash
$ yum install -y mysql-connector-java
$ yum install cloudera-manager-server -y
```

初始化数据库：

```
/usr/share/cmf/schema/scm_prepare_database.sh mysql --host bbd01 --port 3316 cmf cmf CMF1@password
```

启动：

```bash
$ systemctl start cloudera-scm-server
$ systemctl enable cloudera-scm-server
$ systemctl status cloudera-scm-server
```



## Agent

agent 安装在 bbd01 - bbd05 中：

```bash
$ yum install -y mysql-connector-java
$ yum install cloudera-manager-agent -y
$ sed -i "s/server_host=localhost/server_host=bbd01/g" /etc/cloudera-scm-agent/config.ini
```

启动：

```bash

$ systemctl start cloudera-scm-agent
$ systemctl enable cloudera-scm-agent
$ systemctl status cloudera-scm-agent
```





![image-20200915171714006](/Users/jiyouxu/Library/Application Support/typora-user-images/image-20200915171714006.png)





