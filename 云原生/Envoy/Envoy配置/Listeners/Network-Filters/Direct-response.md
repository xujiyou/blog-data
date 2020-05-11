# Direct response

配置也很简单：

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
      - name: envoy.filters.network.direct_response
        typed_config:
         "@type": type.googleapis.com/envoy.config.filter.network.direct_response.v2.Config
         response:
           inline_string: "hello world"
```

对于所有 tcp 访问，都会返回 hello world。

怎么访问那，可以用 curl，因为 http 是建立在 tcp 之上的：

```
$ curl http://localhost:82
curl: (56) Recv failure: Connection reset by peer
hello world
```

虽然看到了一个错误，但是还是返回我们想要的 hello world。

或者使用 golang 写一个 tcp 客户端：

```go
package main

import (
	"bufio"
	"fmt"
	"log"
	"net"
)

func main() {
	conn, err := net.Dial("tcp", "127.0.0.1:82")
	if err != nil {
		log.Fatal(err)
	}
	_,_ = fmt.Fprintln(conn)
	result, _ := bufio.NewReader(conn).ReadString('\n')
	log.Println(result)
}
```

运行客户端，并查看结果：

```bash
$ go run client.go 
2020/04/27 14:24:54 hello world
```

