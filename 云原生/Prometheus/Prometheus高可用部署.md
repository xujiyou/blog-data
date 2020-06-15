# Prometheus 高可用部署

先来学习下 Prometheus 的储存，然后实战怎么高可用部署！

参考 prometheus-book：https://yunlzheng.gitbook.io/prometheus-book/part-ii-prometheus-jin-jie/readmd/prometheus-local-storage



## 储存

Prometheus内置了一个基于本地存储的时间序列数据库。在Prometheus设计上，使用本地存储可以降低Prometheus部署和管理的复杂度同时减少高可用（HA）带来的复杂性。 在默认情况下，用户只需要部署多套Prometheus，采集相同的Targets即可实现基本的HA。同时由于Promethus高效的数据处理能力，单个Prometheus Server基本上能够应对大部分用户监控规模的需求。

当然本地存储也带来了一些不好的地方，首先就是数据持久化的问题，特别是在像Kubernetes这样的动态集群环境下，如果Promthues的实例被重新调度，那所有历史监控数据都会丢失。 其次本地存储也意味着Prometheus不适合保存大量历史数据(一般Prometheus推荐只保留几周或者几个月的数据)。最后本地存储也导致Prometheus无法进行弹性扩展。为了适应这方面的需求，Prometheus提供了remote_write和remote_read的特性，支持将数据存储到远端和从远端读取数据。通过将监控与数据分离，Prometheus能够更好地进行弹性扩展。

除了本地存储方面的问题，由于Prometheus基于Pull模型，当有大量的Target需要采样本时，单一Prometheus实例在数据抓取时可能会出现一些性能问题，联邦集群的特性可以让Prometheus将样本采集任务划分到不同的Prometheus实例中，并且通过一个统一的中心节点进行聚合，从而可以使Prometheuse可以根据规模进行扩展。

除了讨论Prometheus自身的高可用，Alertmanager作为Promthues体系中的告警处理中心，本章的最后部分会讨论如何实现Alertmanager的高可用部署。



## 本地储存



## 单机安装

在 CentOS 7 上使用二进制部署，使用 Systemd 进行管理。

添加用户组及用户：

```bash
$ sudo groupadd --system prometheus
$ sudo useradd -s /sbin/nologin --system -g prometheus prometheus
```

添加目录：

```bash
$ sudo mkdir /data/prometheus
$ sudo chown -R prometheus:prometheus /data/prometheus/
$ for i in rules rules.d files_sd; do sudo mkdir -p /etc/prometheus/${i}; done
$ for i in rules rules.d files_sd; do sudo chown -R prometheus:prometheus /etc/prometheus/${i}; done
$ for i in rules rules.d files_sd; do sudo chmod -R 775 /etc/prometheus/${i}; done
```

去官网下载 Prometheus 二进制包。移动二进制文件到 PATH：

```bash
$ sudo mv prometheus promtool tsdb /usr/local/bin/
```

移动配置文件到配置目录：

```bash
$ sudo mv prometheus.yml  /etc/prometheus/prometheus.yml
$ sudo mv consoles/ console_libraries/ /etc/prometheus/
```

移动完成后，就可以删掉二进制包了。

修改配置文件，可以修改自定义配置 ：

```bash
$ sudo vim /etc/prometheus/prometheus.yml
```

添加 service 文件：

```bash
$ sudo vim /usr/lib/systemd/system/prometheus.service 
```

内容如下：

```
[Unit]
Description=Prometheus
Documentation=https://prometheus.io/docs/introduction/overview/
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
Environment="GOMAXPROCS=16"
User=prometheus
Group=prometheus
ExecReload=/bin/kill -HUP $MAINPID
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/data/prometheus \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.external-url=

SyslogIdentifier=prometheus
Restart=always

[Install]
WantedBy=multi-user.target
```

因为我这里 CPU 为 16 核，所以设置了环境变量 `GOMAXPROCS=16`

启动：

```bash
$ sudo systemctl daemon-reload
$ sudo systemctl start prometheus
$ sudo systemctl enable prometheus
```

查看状态：

```
$ sudo systemctl status prometheus
```

浏览器打开界面：http://drift-1:9090/graph



## 将数据储存到 Elasticsearch

安装一个单机 Elasticsearch：

```
$ sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```

编写 repo：

```
sudo vim /etc/yum.repos.d/elasticsearch.repo
```

内容如下：

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

安装 ES：

```
sudo yum install --enablerepo=elasticsearch elasticsearch
```

启动：

```bash
$ sudo systemctl daemon-reload
$ sudo systemctl start elasticsearch
$ sudo systemctl enable elasticsearch
```

安装 Metricbeat：

```bash
$ sudo yum install metricbeat --enablerepo=elasticsearch
$ sudo systemctl enable metricbeat
$ sudo systemctl start metricbeat
```







