# kafka 协议代理

Envoy 代理 kafka 的流量。

Envoy 配置如下：

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
      - name: envoy.filters.network.kafka_broker
        typed_config:
          "@type": type.googleapis.com/envoy.config.filter.network.kafka_broker.v2alpha1.KafkaBroker
          stat_prefix: exampleprefix
      - name: envoy.filters.network.tcp_proxy
        typed_config:
          "@type": type.googleapis.com/envoy.config.filter.network.tcp_proxy.v2.TcpProxy
          stat_prefix: tcp
          cluster: localkafka
  clusters:
  - name: localkafka
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    hosts:
    - socket_address:
        address: 127.0.0.1 # Kafka broker's host.
        port_value: 9092   # Kafka broker's port.
```

然后修改 kafaka 的配置文件 `config\server.properties`

```
listeners=PLAINTEXT://127.0.0.1:9092
advertised.listeners=PLAINTEXT://127.0.0.1:82
```



测试：

```bash
$ bin/kafka-topics.sh --create --bootstrap-server 127.0.0.1:82 --replication-factor 1 --partitions 1 --topic my-topicbin/kafka-topics.sh --create --bootstrap-server 127.0.0.1:82 --replication-factor 1 --partitions 1 --topic my-topic
```

