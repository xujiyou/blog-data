# lua 脚本过滤器

HTTP Lua 过滤器允许 [Lua](https://www.lua.org/) 脚本在请求和响应流期间运行，以自定义任何想要的功能！

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
            - name: envoy.filters.http.lua
              typed_config:
                "@type": type.googleapis.com/envoy.config.filter.http.lua.v2.Lua
                inline_code: |
                  local mylibrary = require("mylibrary")

                  function envoy_on_request(request_handle)
                    request_handle:headers():add("foo", mylibrary.foobar())
                  end
                  function envoy_on_response(response_handle)
                    body_size = response_handle:body():length()
                    response_handle:headers():add("foo", mylibrary.foobar())
                    response_handle:headers():add("response-body-size", tostring(body_size))
                  end
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

mylibrary.lua 如下：

```lua
M = {}

function M.foobar()
    return "bar"
end

return M
```

服务启动后，测试访问：

```bash
$ curl -v http://localhost:82/ping
```

响应如下：

```
* About to connect() to localhost port 82 (#0)
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 82 (#0)
> GET /ping HTTP/1.1
> User-Agent: curl/7.29.0
> Host: localhost:82
> Accept: */*
> 
< HTTP/1.1 200 OK
< content-type: application/json; charset=utf-8
< date: Sun, 26 Apr 2020 05:06:14 GMT
< content-length: 18
< x-envoy-upstream-service-time: 2
< foo: bar
< response-body-size: 18
< server: envoy
< 
* Connection #0 to host localhost left intact
{"message":"pong"}
```

注意看响应头。