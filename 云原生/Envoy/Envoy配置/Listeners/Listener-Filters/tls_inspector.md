# TLS Inspector

这也是一个专门用于统计的过滤器。

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
    listener_filters:
      - name: "envoy.filters.listener.tls_inspector"
        typed_config: {}
    filter_chains:
    - filters:
      - name: envoy.filters.network.http_connection_manager
        typed_config:
          "@type": type.googleapis.com/envoy.config.filter.network.http_connection_manager.v2.HttpConnectionManager
          codec_type: auto
          stat_prefix: ingress_http
          access_log:
            name: envoy.file_access_log
            typed_config:
              "@type": type.googleapis.com/envoy.config.accesslog.v2.FileAccessLog
              path: /dev/stdout
          route_config:
            name: local_route
            virtual_hosts:
            - name: service
              domains:
              - "*"
              routes:
              - match:
                  prefix: "/"
                route:
                  cluster: ping-service
          http_filters:
            - name: envoy.filters.http.router
  clusters:
  - name: ping-service
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    load_assignment:
      cluster_name: ping-service
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: 127.0.0.1
                port_value: 9090
```



统计内容如下：

```
tls_inspector.alpn_found: 0
tls_inspector.alpn_not_found: 0
tls_inspector.client_hello_too_large: 0
tls_inspector.connection_closed: 0
tls_inspector.read_error: 0
tls_inspector.sni_found: 0
tls_inspector.sni_not_found: 0
tls_inspector.tls_found: 0
tls_inspector.tls_not_found: 1
```

