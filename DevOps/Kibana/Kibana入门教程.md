# Kibana 入门教程

首先第一步，先安装，使用 RPM 安装。

```bash
$ rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
$ sudo vim /etc/yum.repos.d/kibana.repo
[kibana-7.x]
name=Kibana repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
$ sudo yum install kibana
```

修改配置文件 `/etc/kibana/kibana.yml`

```
server.host: "0.0.0.0"
```

启动：

```bash
$ sudo /bin/systemctl daemon-reload
$ sudo /bin/systemctl enable kibana.service
$ sudo systemctl start kibana.service
```

