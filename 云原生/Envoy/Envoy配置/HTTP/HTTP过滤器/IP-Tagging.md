# IP Tagging

给 IP 打标签。

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
            - name: envoy.filters.http.ip_tagging
              typed_config:
                "@type": type.googleapis.com/envoy.config.filter.http.ip_tagging.v2.IPTagging
                request_type: BOTH
                ip_tags:
                  - ip_tag_name: one
                    ip_list:
                      - address_prefix: 127.0.0.1
                        prefix_len: 32
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

IP 地址 127.0.0.1 被打上了 one 标签。

这里来测试一下。

运行三次下面这个命令：

```bash
$ curl -v http://127.0.0.1:82/ping
```

再运行一次：

```bash
$ curl -v http://172.20.20.162:82/ping
```

然后再访问：`http://172.20.20.162:8081/stats` 界面，会发现其中有以下数据：

```
http.ingress_http.ip_tagging.no_hit: 1
http.ingress_http.ip_tagging.one.hit: 3
http.ingress_http.ip_tagging.total: 4
```

表示被打了 one 标签的 IP 被访问了 3 次，一共被访问了 4 次，访问未打标签的 IP 一次。







