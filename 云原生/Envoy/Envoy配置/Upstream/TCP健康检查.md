# TCP 健康检查

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
      - name: envoy.filters.network.tcp_proxy
        typed_config:
          "@type": type.googleapis.com/envoy.config.filter.network.tcp_proxy.v2.TcpProxy
          stat_prefix: tcp
          cluster: tcp-cluster
  clusters:
  - name: tcp-cluster
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    hosts:
    - socket_address:
        address: 127.0.0.1
        port_value: 1234
    health_checks:
      timeout: 1s
      interval: 10s
      unhealthy_threshold: 9
      healthy_threshold: 0
      tcp_health_check:
        send: {text: '41'}
        receive: {text: '42'}
```

服务端代码如下：

```go
package main

import (
	"fmt"
	"log"
	"net"
	"os"
)

func main() {
	l, err := net.Listen("tcp", "127.0.0.1:1234")
	if err != nil {
		fmt.Println("Error listening:", err)
		os.Exit(1)
	}
	defer l.Close()
	log.Println("Listening on 127.0.0.1:1234")

	for {
		conn, err := l.Accept()
		if err != nil {
			fmt.Println("Error accepting: ", err)
			os.Exit(1)
		}

		log.Printf("Received message %s -> %s \n", conn.RemoteAddr(), conn.LocalAddr())

		go doServerStuff(conn)
	}
}

func doServerStuff(conn net.Conn) {
	for {
		buf := make([]byte, 512)
		len, err := conn.Read(buf)
		if err != nil {
			log.Println("END reading", err.Error())
			return //终止程序
		}
		log.Printf("Received data: %v", string(buf[:len]))

		if string(buf) == "A" {
			_, _ = conn.Write([]byte("B"))
		} else {
			_, _ = conn.Write(buf)
		}
	}
}
```

