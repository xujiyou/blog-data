# Envoy 集成 Jaeger

使用 GetEnvoy 无效，只能使用 docker-compose 来测试，亲测！！！

关于 tracing，Envoy 支持以下几种：

- *envoy.tracers.lightstep*
- *envoy.tracers.zipkin*
- *envoy.tracers.dynamic_ot*
- *envoy.tracers.datadog*
- *envoy.tracers.opencensus*
- *envoy.tracers.xray*

详情请见：https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/trace/v2/http_tracer.proto#envoy-api-msg-config-trace-v2-tracing-http

下面来学习如何使用，这里只贴一下 Envoy 配置：

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
          tracing:
            provider:
              name: envoy.tracers.zipkin
              typed_config:
                "@type": type.googleapis.com/envoy.config.trace.v2.ZipkinConfig
                collector_cluster: jaeger
                collector_endpoint: "/api/v2/spans"
                shared_span_context: false
                collector_endpoint_version: HTTP_JSON
          codec_type: auto
          stat_prefix: ingress_http
          route_config:
            name: service1_route
            virtual_hosts:
            - name: service1
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
  - name: jaeger
    connect_timeout: 1s
    type: strict_dns
    lb_policy: round_robin
    load_assignment:
      cluster_name: jaeger
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: jaeger
                port_value: 9411
admin:
  access_log_path: "/dev/null"
  address:
    socket_address:
      address: 0.0.0.0
      port_value: 20001
```

注意其中的 `tracing`，关于 jaeger ：https://www.jaegertracing.io/docs/1.17/getting-started/

如果想测试，可以把 https://github.com/xujiyou/envoy-examples/tree/master/jaeger 中的代码下载下来，然后一条命令启动：

```bash
$ docker-compose up --build
```

对服务进行访问：

```bash
$ curl http://localhost:82/hello
```

查看 jaeger UI：http://fueltank-1:16686/

