# 外部认证

创建项目 authz ，并使用 `go mod init authz` 初始化。

先写个 认证服务端，使用 go-gin 框架，命名为 authz.go ：

```go
package main

import "github.com/gin-gonic/gin"

func main() {
	r := gin.Default()
	r.GET("/authz/*action", func(c *gin.Context) {
		authzHeader := c.Request.Header.Get("Authorization")
		if authzHeader == "my-token" {
			c.Writer.WriteHeader(200)
		} else {
			c.Writer.WriteHeader(401)
		}
	})
	_ = r.Run("0.0.0.0:6060")
}
```

这个服务接收所有 `/authz/*` 路径下的请求。

然后写应答服务端，也是用的 `go-gin` 框架，命名为 server.go :

```go
package main

import "github.com/gin-gonic/gin"

func main() {
	r := gin.Default()
	r.GET("/ping", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "pong",
		})
	})
	_ = r.Run("0.0.0.0:9090")
}
```

然后编写 Envoy 的配置文件，命名为 envoy-config.yaml :

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
            - name: envoy.filters.http.ext_authz
              typed_config:
                "@type": type.googleapis.com/envoy.config.filter.http.ext_authz.v2.ExtAuthz
                http_service:
                  path_prefix: /authz
                  server_uri:
                    uri: 127.0.0.1:6060
                    cluster: ext-authz
                    timeout: 0.25s
                failure_mode_allow: false
                include_peer_certificate: true
            - name: envoy.filters.http.router
  clusters:
  - name: ext-authz
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    load_assignment:
      cluster_name: ext-authz
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

然后分别在不同的客户端启动认证服务端，应答服务端和 Envoy：

```bash
go run authz.go
go run server.go
getenvoy run standard:1.14.1 -- --config-path ./envoy-config.yaml
```

然后测试，未通过认证的测试：

```bash
$ curl -L http://localhost:82/ping -H "Authorization: my-token1"
```

通过认证的测试：

```bash
$ curl -L http://localhost:82/ping -H "Authorization: my-token"
```

需要加 -L 选项 curl 才能处理 301 响应。