# Reids 流量代理

Envoy 配置：

```yaml
admin:
  access_log_path: /dev/stdout
  address:
    socket_address:
      address: 0.0.0.0
      port_value: 8081

node:
  cluster: hello-service
  id: node1

static_resources:
  listeners:
  - address:
      socket_address:
        address: 0.0.0.0
        port_value: 82
    filter_chains:
    - filters:
      - name: envoy.filters.network.redis_proxy
        typed_config:
          "@type": type.googleapis.com/envoy.config.filter.network.redis_proxy.v2.RedisProxy
          stat_prefix: redis-stat
          settings:
            op_timeout: 3s
          prefix_routes:
            routes:
              - prefix: ""
                cluster: redis
  clusters:
  - name: redis
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    hosts:
    - socket_address:
        address: 127.0.0.1
        port_value: 6379
```

注意

- `envoy.filters.network.redis_proxy` 过滤器必须是最后一个过滤器
-  `settings.op_timeout` 是必须要配置

测试访问：

```bash
$ redis-cli -h 127.0.0.1 -p 82
```

