# tap 过滤器

这个过滤器用于收集请求和响应数据！

envoy 配置如下：

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
            - name: envoy.filters.http.tap
              typed_config:
                "@type": type.googleapis.com/envoy.config.filter.http.tap.v2alpha.Tap
                common_config:
                  static_config:
                    match_config:
                      any_match: true
                    output_config:
                      sinks:
                        - file_per_tap:
                            path_prefix: /tmp/
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

这个配置会记录所有请求和响应的数据。

访问过后，会记录到 