# CORS

为 Envoy 添加以下配置：

```yaml
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
          route_config:
            name: service1_route
            virtual_hosts:
            - name: service1
              cors:
                allow_origin_string_match:
                  safe_regex:
                    google_re2: {}
                    regex:  ".*"
                allow_methods: "POST, GET, OPTIONS"
                allow_headers: "Content-Type, Authenticate"
              domains:
              - "*"
              routes:
              - match:
                  prefix: "/"
                route:
                  cluster: hello-v1
                decorator:
                  operation: checkAvailability
          http_filters:
          - name: envoy.filters.http.cors
            typed_config: {}
          - name: envoy.filters.http.router
  clusters:
  - name: hello-v1
    connect_timeout: 0.250s
    type: strict_dns
    lb_policy: round_robin
    load_assignment:
      cluster_name: hello-v1
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: 127.0.0.1
                port_value: 8081

admin:
  access_log_path: "/dev/null"
  address:
    socket_address:
      address: 0.0.0.0
      port_value: 20001
```

注意这里 `cors` 和 `envoy.filters.http.cors` 要同时设置才行！！！

`google_re2` 是必须要写上的。

运行 Envoy。

然后打开最新版 chrome 浏览器的 console 进行测试（curl 测试不了，亲测）：

```js
fetch('http://fueltank-1:82/hello').then(function(response) {
    console.log(response.text());
})
```

