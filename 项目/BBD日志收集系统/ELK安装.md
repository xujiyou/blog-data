

搭建一套 Syslog-ng + ELK 系统收集并分析交换机日志。



## JDK 安装

先安装好 JDK 8，去 Oracle 官网下载 RPM 包，然后本地安装：

```bash
$ rpm --install jdk-8u261-linux-x64.rpm
```

设置 JAVA_HOME，在 /etc/profile 底部写入：

```bash
export JAVA_HOME=/usr/java/jdk1.8.0_261-amd64
```

使之生效：

```bash
$ source /etc/profile
```



## Elasticsearch 安装

安装 Elasticsearch，Elasticsearch 版本：7.8.0

参考：https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html

```bash
$ rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```

编辑 `/etc/yum.repos.d/elasticsearch.repo`

```
[elasticsearch]
name=Elasticsearch repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=0
autorefresh=1
type=rpm-md
```

安装：

````bash
$ yum install --enablerepo=elasticsearch elasticsearch
````

修改配置文件 `/etc/elasticsearch/elasticsearch.yml`

```yaml
cluster.name: security
node.name: security-1
path.data: /data1/es,/data2/es,/data3/es,/data4/es
path.logs: /data3/es-log
network.host: 10.28.92.11
discovery.seed_hosts: ["10.28.92.11", "10.28.93.11"]
cluster.initial_master_nodes: ["security-1", "IDS-transfer"]
```

创建目录：

```bash
$ mkdir /data3/es-log
$ mkdir /data1/es
$ mkdir /data2/es
$ mkdir /data3/es
$ mkdir /data4/es
$ chown -R elasticsearch:elasticsearch /data3/es-log
$ chown -R elasticsearch:elasticsearch /data1/es
$ chown -R elasticsearch:elasticsearch /data2/es
$ chown -R elasticsearch:elasticsearch /data3/es
$ chown -R elasticsearch:elasticsearch /data4/es
```

启动：

```bash
$ systemctl enable elasticsearch
$ systemctl start elasticsearch
```

检查状态：

```bash
$ systemctl status elasticsearch
$ curl http://10.28.92.11:9200
```

Elasticsearch 使用了 4 块磁盘，理论上储存的最大数据量为 12 T左右。



## Kibana 安装

Kibana 版本：7.8.0

参考：https://www.elastic.co/guide/en/kibana/current/rpm.html

````bash
$ rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
````

创建 `/etc/yum.repos.d/kibana.repo`

```
[kibana-7.x]
name=Kibana repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```

安装：

```bash
$ yum install kibana
```

修改配置文件 `/etc/kibana/kibana.yml` :

```
server.host: "0.0.0.0"
elasticsearch.hosts: ["http://10.28.92.11:9200"]
```

启动：

```bash
$ systemctl enable kibana
$ systemctl start kibana
```

查看状态：

```bash
$ systemctl status kibana
```

防火墙打开端口：

```bash
$ firewall-cmd --zone=public --add-port=5601/tcp --permanent
$ systemctl reload firewalld
```

浏览器打开：http://10.28.92.11:5601/



## syslog-ng 安装

参考：https://www.syslog-ng.com/community/b/blog/posts/installing-latest-syslog-ng-on-rhel-and-other-rpm-distributions

````bash
$ wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
$ rpm -Uvh epel-release-latest-8.noarch.rpm
$ cd /etc/yum.repos.d/
$ wget https://copr.fedorainfracloud.org/coprs/czanik/syslog-ng328/repo/epel-8/czanik-syslog-ng328-epel-8.repo
$ yum install syslog-ng
````

配置文件是 `/etc/syslog-ng/syslog-ng.conf`，备份默认的配置文件：

```bash
$ mv /etc/syslog-ng/syslog-ng.conf /etc/syslog-ng/syslog-ng.conf.bck
```

配置文件为 `/etc/syslog-ng/syslog-ng.conf` 。

启动：

```bash
$ systemctl enable syslog-ng
$ systemctl start syslog-ng
```

查看状态：

```bash
$ systemctl status syslog-ng
$ lsof -i:514
$ netstat -aun
```

打开防火墙端口：

```bash
$ firewall-cmd --zone=public --add-port=514/tcp --permanent
$ firewall-cmd --zone=public --add-port=514/udp --permanent
$ firewall-cmd --zone=public --add-port=5140/udp --permanent
$ systemctl reload firewalld
```



## Logstash 安装

参考：https://www.elastic.co/guide/en/logstash/7.8/installing-logstash.html

版本：7.8.0

```bash
$ rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```

创建 `/etc/yum.repos.d/logstash.repo`

```
[logstash-7.x]
name=Elastic repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```

安装：

```bash
$ yum install logstash
```

修改配置文件 `/etc/logstash/logstash.yml`

```
node.name: security-1
path.data: /data3/logstash
pipeline.workers: 24
pipeline.ordered: auto
path.config: /etc/logstash/conf.d/*.conf
config.reload.automatic: true
config.reload.interval: 10s
dead_letter_queue.enable: true
dead_letter_queue.max_bytes: 2048mb
http.host: "0.0.0.0"
path.logs: /data3/logstash-log
```

启动 Logstash：

```bash
$ systemctl enable logstash
$ systemctl start logstash
```

查看状态：

```bash
$ systemctl status logstash
```

在 `/etc/logstash/conf.d/` 中添加规则文件。











