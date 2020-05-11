# mongo 流量代理

Envoy 的配置如下：

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
      - name: envoy.filters.network.mongo_proxy
        typed_config:
          "@type": type.googleapis.com/envoy.config.filter.network.mongo_proxy.v2.MongoProxy
          stat_prefix: my_mongo
      - name: envoy.filters.network.tcp_proxy
        typed_config:
          "@type": type.googleapis.com/envoy.config.filter.network.tcp_proxy.v2.TcpProxy
          stat_prefix: tcp
          cluster: mongo
  clusters:
  - name: mongo
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    hosts:
    - socket_address:
        address: 127.0.0.1
        port_value: 27017
```

测试链接：

```bash
$ mongo 127.0.0.1:82
```

