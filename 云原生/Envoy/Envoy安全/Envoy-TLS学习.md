# Envoy TLS 学习

学习单向认证，双向认证，对 Downstream 和 UpStream 的认证。

## Envoy 单向认证

Envoy 对 Downstream 进行单向认证：

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
    - transport_socket:
        name: envoy.transport_sockets.tls
        typed_config:
          "@type": type.googleapis.com/envoy.api.v2.auth.DownstreamTlsContext
          common_tls_context:
            tls_certificates:
              certificate_chain: { "filename": "/home/admin/k8s-cluster/envoy/ssl/cert/server.crt" }
              private_key: { "filename": "/home/admin/k8s-cluster/envoy/ssl/cert/server.key" }
            validation_context:
              trusted_ca:
                filename: /home/admin/k8s-cluster/envoy/ssl/cert/ca.crt
      filters:
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

```

测试：

```bash
$ curl --cacert /home/admin/k8s-cluster/envoy/ssl/cert/ca.crt https://fueltank-1:82/ping
```

不加 ca 证书也可以访问：

```
$ curl --cacert /home/admin/k8s-cluster/envoy/ssl/cert/ca.crt https://fueltank-1:82/ping
```





## 双向认证

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
      - name: envoy.filters.network.client_ssl_auth
        typed_config:
          "@type": type.googleapis.com/envoy.config.filter.network.client_ssl_auth.v2.ClientSSLAuth
          auth_api_cluster: cert-service
          stat_prefix: cert
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
      transport_socket:
        name: envoy.transport_sockets.tls
        typed_config:
          "@type": type.googleapis.com/envoy.api.v2.auth.DownstreamTlsContext
          require_client_certificate: true # 开启双向认证，要求客户端提供证书
          common_tls_context:
            tls_certificates:
              certificate_chain:
                filename: /home/admin/k8s-cluster/envoy/ssl/cert/server.crt
              private_key:
                filename: /home/admin/k8s-cluster/envoy/ssl/cert/server.key
            validation_context:
              trusted_ca:
                filename: /home/admin/k8s-cluster/envoy/ssl/cert/ca.crt
              trust_chain_verification: ACCEPT_UNTRUSTED # 必须要加的
  clusters:
  - name: cert-service
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    load_assignment:
      cluster_name: cert-service
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
    transport_socket: # 对服务端进行单向认证
      name: envoy.transport_sockets.tls
      typed_config: {}
```



这里 golang 的自己开发的服务端 ping-service 的代码如下，命名为 main.go ：

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
	_ = r.RunTLS("0.0.0.0:9090", "/home/admin/k8s-cluster/envoy/ssl/cert/server.crt", "/home/admin/k8s-cluster/envoy/ssl/cert/server.key")
}
```

因为这里 golang 的服务端开启了单向认证，所以要在对应的 `cluster` 中添加 `envoy.transport_sockets.tls` ，因为是单向认证，所以这里内容可以置空。



### Envoy 对客户端的双向认证



因为这里我使用了 `envoy.filters.network.client_ssl_auth` 这个过滤器来验证客户端证书。所以要自研 `cert-service` 服务。其代码如下，命名为 server.go ：

```go
package main

import "github.com/gin-gonic/gin"

func main() {
	r := gin.Default()
	r.GET("/v1/certs/list/approved", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"certificates": []map[string]string{
				{
					"fingerprint_sha256": "a45e46487977c62cd60a3a4e8ec044f8fe16115bd1c8f5cddf3d99f82dc864a7",
				},
			},
		})
	})
	_ = r.Run("0.0.0.0:6060")
}
```

这里 API 的地址和响应的格式是固定的，必须这么配置，Envoy 才能得到对应的响应信息，响应格式如下：

```json
{
    "certificates": [
        {
            "fingerprint_sha256": "a45e46487977c62cd60a3a4e8ec044f8fe16115bd1c8f5cddf3d99f82dc864a7"
        }
    ]
}
```

这里 `fingerprint_sha256` 中的内容是客户端证书的摘要，其生成方式如下：

```bash
$ openssl x509 -in client.crt -outform DER | openssl dgst -sha256 | cut -d" " -f2
a45e46487977c62cd60a3a4e8ec044f8fe16115bd1c8f5cddf3d99f82dc864a7
```

如果客户端使用的证书的摘要和这里规定的不匹配，则客户端会访问失败！

分别启动两个服务端：

```bash
$ go run server.go 
```

```bash
$ go run main.go
```

然后启动 Envoy：

```bash
$ sudo getenvoy run standard:1.14.1 -- --config-path ./envoy-config.yaml
```

客户端访问：

```bash
$ curl --cert ./server.crt --key ./server.key https://fueltank-1:82/ping
```

如果证书对了是可以访问成功的，证书不对，则访问失败

查看 Envoy 的统计信息：

```
auth.clientssl.cert.auth_digest_match: 4
auth.clientssl.cert.auth_digest_no_match: 1
auth.clientssl.cert.auth_ip_white_list: 0
auth.clientssl.cert.auth_no_ssl: 0
auth.clientssl.cert.total_principals: 1
auth.clientssl.cert.update_failure: 0
auth.clientssl.cert.update_success: 2
```



## SNI

SNI（Server Name Indication）是 TLS 的扩展，用来解决一个服务器拥有多个域名的情况。

见教程：https://www.envoyproxy.io/docs/envoy/latest/faq/configuration/sni#faq-how-to-setup-sni

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
    listener_filters:
      - name: "envoy.filters.listener.tls_inspector" # 必须要配置
        typed_config: {}
    filter_chains:
      - filter_chain_match:
          server_names: ["fueltank-1"]
        transport_socket:
          name: envoy.transport_sockets.tls
          typed_config:
            "@type": type.googleapis.com/envoy.api.v2.auth.DownstreamTlsContext
            require_client_certificate: true
            common_tls_context:
              tls_certificates:
                certificate_chain:
                  filename: /home/admin/k8s-cluster/envoy/ssl/cert/server.crt
                private_key:
                  filename: /home/admin/k8s-cluster/envoy/ssl/cert/server.key
              validation_context:
                trusted_ca:
                  filename: /home/admin/k8s-cluster/envoy/ssl/cert/ca.crt
                trust_chain_verification: ACCEPT_UNTRUSTED
        filters:
        - name: envoy.filters.network.client_ssl_auth
          typed_config:
            "@type": type.googleapis.com/envoy.config.filter.network.client_ssl_auth.v2.ClientSSLAuth
            auth_api_cluster: cert-service
            stat_prefix: cert
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
  - name: cert-service
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    load_assignment:
      cluster_name: cert-service
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
    transport_socket:
      name: envoy.transport_sockets.tls
      typed_config: {}
```

