# MySQL 流量代理

envoy 配置文件：

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
      - name: envoy.filters.network.mysql_proxy
        typed_config:
          "@type": type.googleapis.com/envoy.config.filter.network.mysql_proxy.v1alpha1.MySQLProxy
          stat_prefix: mysql
      - name: envoy.filters.network.tcp_proxy
        typed_config:
          "@type": type.googleapis.com/envoy.config.filter.network.tcp_proxy.v2.TcpProxy
          stat_prefix: tcp
          cluster: mysql
  clusters:
  - name: mysql
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    hosts:
    - socket_address:
        address: 127.0.0.1
        port_value: 3506
```

测试访问：

```bash
$ mysql -u root -h 127.0.0.1 -P 82 -p
```

