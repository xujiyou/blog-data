# Buffer

过滤器名为 `envoy.filters.http.buffer`

示例：

首先创建一个项目，名为 buffer，然后编写一个 API 服务，命名为 main.go：

````go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

type Login struct {
	User     string `form:"user" json:"user" xml:"user"  binding:"required"`
	Password string `form:"password" json:"password" xml:"password" binding:"required"`
}

func main() {
	router := gin.Default()

	// Example for binding JSON ({"user": "manu", "password": "123"})
	router.POST("/loginJSON", func(c *gin.Context) {
		var json Login
		if err := c.ShouldBindJSON(&json); err != nil {
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
			return
		}

		if json.User != "manu" || json.Password != "123" {
			c.JSON(http.StatusUnauthorized, gin.H{"status": "unauthorized"})
			return
		}

		c.JSON(http.StatusOK, gin.H{"status": "you are logged in"})
	})

	// Listen and serve on 0.0.0.0:8081
	_ = router.Run(":8081")
}
````

启动服务：

```bash
$ go mod init buffer
$ go run main.go
```

编写 envoy 配置文件：

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
          - name: envoy.filters.http.buffer
            typed_config:
              "@type": type.googleapis.com/envoy.config.filter.http.buffer.v2.Buffer
              max_request_bytes: 1
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

这里设置了 `max_request_bytes` 为1，即限制 `Content-Length` 最大为 1

启动 Envoy：

```bash
$ sudo getenvoy run standard:1.14.1 -- --config-path ./envoy-config.yaml
```

测试访问：

```bash
$ curl -v -X POST http://localhost:82/loginJSON -H 'content-type: application/json' -d '{"user": "manu", "password": "123"}'
```

返回如下：

```
* About to connect() to localhost port 82 (#0)
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 82 (#0)
> POST /loginJSON HTTP/1.1
> User-Agent: curl/7.29.0
> Host: localhost:82
> Accept: */*
> content-type: application/json
> Content-Length: 35
> 
* upload completely sent off: 35 out of 35 bytes
< HTTP/1.1 413 Payload Too Large
< content-length: 17
< content-type: text/plain
< date: Fri, 24 Apr 2020 07:59:21 GMT
< server: envoy
< connection: close
< 
* Closing connection 0
Payload Too Large
```

出现了 413 错误，表示请求数据过大

如果提高 `max_request_bytes` 到 36 ，就可以访问了。











