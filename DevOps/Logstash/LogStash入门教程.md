#  LogStash 教程

首先安装，yum 安装：

```bash
$ rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
$ vim /etc/yum.repos.d/logstash.repo
[logstash-7.x]
name=Elastic repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
$ sudo yum install logstash
```

启动 Logstash

```bash
$ sudo systemctl start logstash
```

安装目录：`/usr/share/logstash`，配置文件目录：`/etc/logstash`

启动配置文件：`vim /etc/logstash/startup.options`

另外，看看 `/etc/logstash/pipelines.yml` 这个文件：

```
# This file is where you define your pipelines. You can define multiple.
# For more information on multiple pipelines, see the documentation:
#   https://www.elastic.co/guide/en/logstash/current/multiple-pipelines.html

- pipeline.id: main
  path.config: "/etc/logstash/conf.d/*.conf"
```

这个文件说明了logstash 会在启动时加载 `/etc/logstash/conf.d/` 目录下的所有以 .conf 结尾的配置文件，所以线上的所有配置文件放在这个目录下，重启 logstash 后，会自动加载新配置。

如果想让 Logstash 在运行时自动加载修改的配置文件，可以修改 `/etc/logstash/logstash.yml` 中的配置：

```
config.reload.automatic: true
```

另外，`/etc/logstash/logstash-sample.conf` 只是一个配置样例。

## First Event

首先执行命令

```
$ sudo bin/logstash -e 'input { stdin { } } output { stdout {} }'
```

等一会之后，logstash 会打印一些日志，看到 Successfully started 之后，可以输入一些东西了：

```
[INFO ] 2019-12-26 18:02:10.266 [Api Webserver] agent - Successfully started Logstash API endpoint {:port=>9601}
hello my logstash
{
    "@timestamp" => 2019-12-26T10:03:01.918Z,
      "@version" => "1",
          "host" => "fueltank-4.cloud.bbdops.com",
       "message" => "hello my logstash"
}
```

## 通过 Logstash 解析日志

logstash 配置由 `input` `filter` `output` 组成。

下面通过一个例子来实践一下。

首先安装 Filebeat:

```
$ curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.5.1-x86_64.rpm
$ sudo rpm -vi filebeat-7.5.1-x86_64.rpm
```

然后修改配置文件 `/etc/filebeat/filebeat.yml` 

```yaml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/messages

output.logstash:
  hosts: ["localhost:5044"]
```

/var/log/messages 中记录的是 syslog-ng 的部分日志。

注意！设置了 output.logstash 之后，应该把 output.elasticsearch 注释掉，否则 filebeat 会启动失败！

完成后启动 filebeat：

```
$ sudo systemctl start filebeat
```

然后再写 Logstash 的配置文件，简单复制下  logstash-sample.conf ，然后改一下：

```
$ sudo cp /etc/logstash/logstash-sample.conf /etc/logstash/conf.d/log-message.conf
$ sudo vim /etc/logstash/conf.d/log-message.conf
```

然后将其修改为以下内容

```
input {
  beats {
    port => 5044
  }
}

output {
  stdout { codec => rubydebug }
}
```

这里就是简单的输出到标准输出

测试配置文件是否正确，执行以下命令：

```bash
$ sudo ./bin/logstash -f /etc/logstash/conf.d/log-message.conf  --config.test_and_exit
```

如果格式正确，将会有以下输出：

```
[INFO ] 2019-12-27 10:28:34.245 [LogStash::Runner] runner - Using config.test_and_exit mode. Config Validation Result: OK. Exiting Logstash
```

如果想重新加载配置文件，可以通过以下命令，而不需要重启 Logstash

```bash
$ sudo ./bin/logstash -f /etc/logstash/conf.d/log-message.conf  --config.reload.automatic
```

或者 Logstash 配置了自动加载已修改的配置文件话，什么都不需要做，验证一下配置文件是否正确即可。

最后看下效果，可以使用 `journalctl` 来查看系统日志：

```bash
$ journalctl -n 200
```

`journalctl` 是一个查看系统日志的工具，他的命令和 vim 一样，-n 参数指定了最新的200行，如果不加-n就要小心了，容易卡死。



### Logstash 输出到 Elasticsearch

修改 log-message.log:

```
input {
    beats {
        port => "5044"
    }
}

output {
    elasticsearch {
        hosts => ["http://localhost:9200"]
        index => "log-message-%{+YYYY.MM.dd}"
    }
}
```

这样就在 Elasticsearch 中看到数据了，可以使用 Kibana 查看。

input 和 output 内可以有多个项。

除了 input 和 output ，还有 filter ，filter 也可以有多个项。

input、 filter、 和 output 都可以定义插件。 