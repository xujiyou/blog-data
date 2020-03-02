# Prometheus 集群

配置 Promeetheus 集群：

```yaml
scrape_configs:
  - job_name: 'federate'
    scrape_interval: 15s

    honor_labels: true
    metrics_path: '/federate'

    params:
      'match[]':
        - '{job="prometheus"}'
        - '{__name__=~"job:.*"}'

    static_configs:
      - targets:
        - 'source-prometheus-1:9090'
        - 'source-prometheus-2:9090'
        - 'source-prometheus-3:9090'
```



## 管理 API

### Health check

```
GET /-/healthy
```

例如：http://localhost:9090/-/healthy

### Readiness check

```
GET /-/ready
```

### Reload

```
PUT  /-/reload
POST /-/reload
```

### Quit

```
PUT  /-/quit
POST /-/quit
```

reload 和 quit 都需要 --web.enable-lifecycle 选项开启。



