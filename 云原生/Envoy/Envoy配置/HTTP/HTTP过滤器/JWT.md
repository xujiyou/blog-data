# JWT 过滤器

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
            - name: envoy.filters.http.jwt_authn
              typed_config:
                "@type": type.googleapis.com/envoy.config.filter.http.jwt_authn.v2alpha.JwtAuthentication
                providers:
                  provider1:
                    issuer: testing@secure.istio.io
                    local_jwks:
                      filename: /var/www/html/jwks.json
                    from_headers:
                      - name: jwt-assertion
                    forward: true
                    forward_payload_header: x-jwt-payload
                rules:
                  - match:
                      prefix: /
                    requires:
                      provider_name: provider1
            - name: envoy.filters.http.router
  clusters:
  - name: ext-jwt
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    load_assignment:
      cluster_name: ext-jwt
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: 127.0.0.1
                port_value: 6060
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

其中，`/var/www/html/jwks.json` 的内容如下：

```json
{ "keys":[ {"e":"AQAB","kid":"DHFbpoIUqrY8t2zpA2qXfCmr5VO5ZEr4RzHU_-envvQ","kty":"RSA","n":"xAE7eB6qugXyCAG3yhh7pkDkT65pHymX-P7KfIupjf59vsdo91bSP9C8H07pSAGQO1MV_xFj9VswgsCg4R6otmg5PV2He95lZdHtOcU5DXIg_pbhLdKXbi66GlVeK6ABZOUW3WYtnNHD-91gVuoeJT_DwtGGcp4ignkgXfkiEm4sw-4sfb4qdt5oLbyVpmW6x9cfa7vs2WTfURiCrBoUqgBo_-4WTiULmmHSGZHOjzwa8WtrtOQGsAFjIbno85jp6MnGGGZPYZbDAa_b3y5u-YpW7ypZrvD8BgtKVjgtQgZhLAGezMt0ua3DRrWnKqTZ0BJ_EyxOGuHJrLsn00fnMQ"}]}
```

jwks.json 是一个公钥，获取方式是：

```bash
$ sudo sh -c "curl https://raw.githubusercontent.com/istio/istio/release-1.5/security/tools/jwt/samples/jwks.json > /var/www/html/jwks.json"
```

获取 JWT 的方式是：

```bash
$ wget https://raw.githubusercontent.com/istio/istio/release-1.5/security/tools/jwt/samples/demo.jwt
```

测试访问：

```bash
$ TOKEN=$(cat demo.jwt)
$ curl http://localhost:82/ping -H "jwt-assertion: $TOKEN"
```

如果上面不指定 `from_headers` 的话，就需要使用下面这种方式：

```bash
$ curl http://localhost:82/ping -H "Authorization: Bearer $TOKEN"
```

