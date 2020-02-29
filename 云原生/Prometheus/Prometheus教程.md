# Prometheus 教程

文档地址：https://prometheus.io/docs/prometheus/latest/getting_started/

### 配置 Prometheus 监控自己

Prometheus 可以收集 http 的监控数据，当然，它也可以监控自己。

查看一下 prometheus.yml ：

```yaml
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['localhost:9090']
```

可以看到在 scrape_configs 中，配置了 job name 和监控目标，一会再学完整的配置。

### 一个例子

进入 $GOPATH/src 目录，再运行一下命令：

```bash
# Fetch the client library code and compile example.
$ git clone https://github.com/prometheus/client_golang.git
$ cd client_golang/examples/random
$ go get -d
$ go build

# Start 3 example targets in separate terminals:
$ ./random -listen-address=:8080
$ ./random -listen-address=:8081
$ ./random -listen-address=:8082
```

这个 client_golang 就是启动一个 http 服务，访问时，会调用 prometheus 的 API，来获取 metrics。

修改 prometheus.yml ：

```yaml
scrape_configs:
  - job_name:       'example-random'

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s

    static_configs:
      - targets: ['localhost:8080', 'localhost:8081']
        labels:
          group: 'production'

      - targets: ['localhost:8082']
        labels:
          group: 'canary'
```

重启 prometheus。

分别访问 http://localhost:8080/metrics, http://localhost:8081/metrics, and http://localhost:8082/metrics.

刷新 http://localhost:9090/graph 

先查询 `rpc_durations_seconds_count` ，再查：

```
avg(rate(rpc_durations_seconds_count[5m])) by (job, service)
```



下一步，创建 prometheus.rules.yml：

```yaml
groups:
- name: example
  rules:
  - record: job_service:rpc_durations_seconds_count:avg_rate5m
    expr: avg(rate(rpc_durations_seconds_count[5m])) by (job, service)
```

修改 prometheus.yml ：

```
rule_files:
  - 'prometheus.rules.yml'
```

重启 prometheus 和上边的那三个服务。

重新打开 http://localhost:9090/graph ，直接搜即可：

```
job_service:rpc_durations_seconds_count:avg_rate5m
```



