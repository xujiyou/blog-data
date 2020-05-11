# ZooKeeper 流量代理

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
      - name: envoy.filters.network.zookeeper_proxy
        typed_config:
          "@type": type.googleapis.com/envoy.config.filter.network.zookeeper_proxy.v1alpha1.ZooKeeperProxy
          stat_prefix: zookeeper
      - name: envoy.filters.network.tcp_proxy
        typed_config:
          "@type": type.googleapis.com/envoy.config.filter.network.tcp_proxy.v2.TcpProxy
          stat_prefix: tcp
          cluster: zookeeper
  clusters:
  - name: zookeeper
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    hosts:
    - socket_address:
        address: 127.0.0.1
        port_value: 2181
```



测试 访问：

```bash
$ ./bin/zkCli.sh -timeout 5000 -server 127.0.0.1:82
```

