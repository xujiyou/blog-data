# Gzip 过滤器

envoy 的配置很简单：

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
            - name: envoy.filters.http.gzip
              typed_config: {}
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

使用以下命令测试：

```bash
$ curl -v http://localhost:82/ping -H "Accept-Encoding: gzip"
```

或：

```bash
$ curl -v http://localhost:82/ping -H "Accept-Encoding: *"
```

访问后，会看到一串乱码，这就是被压缩后的数据，如果想用 curl 查看解压后的数据，可以这样：

```bash
$ curl -v --compressed http://localhost:82/ping -H "Accept-Encoding: gzip"
```



更多注意事项请看：https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/gzip_filter#