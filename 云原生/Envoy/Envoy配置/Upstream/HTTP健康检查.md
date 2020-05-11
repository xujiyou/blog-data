# HTTP 健康检查

配置如下：

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
    health_checks:
      timeout: 1s
      interval: 60s
      unhealthy_threshold: 503
      healthy_threshold: 200
      http_health_check:
        path: /healthcheck
```



服务端如下：

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
	r.GET("/healthcheck", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"healthCheck": "success",
		})
	})
	_ = r.Run("0.0.0.0:9090")
}
```



有以下统计信息：

```
cluster.ping-service.health_check.attempt: 2
cluster.ping-service.health_check.degraded: 0
cluster.ping-service.health_check.failure: 0
cluster.ping-service.health_check.healthy: 1
cluster.ping-service.health_check.network_failure: 0
cluster.ping-service.health_check.passive_failure: 0
cluster.ping-service.health_check.success: 2
cluster.ping-service.health_check.verify_cluster: 0
```

