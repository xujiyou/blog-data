# Envoy 代理自己的服务

通过  [Envoy入门.md](Envoy入门.md) 的学习，已经知道了 Envoy 是怎么一个套路了，下面就来从头到尾来创建一个自己的服务，并用 Envoy 进行代理。

本文使用 `go-restful` 进行构建 web 服务。



## 创建 Web 服务

创建项目，名为 hello-envoy

然后编写 golang 代码，命名为 main.go ：

```go
package main

import (
	"github.com/emicklei/go-restful"
	"io"
	"log"
	"net/http"
)

// This example shows the minimal code needed to get a restful.WebService working.
//
// GET http://localhost:8080/hello

func main() {
	ws := new(restful.WebService)
	ws.Route(ws.GET("/hello").To(hello))
	restful.Add(ws)
	log.Println("server start in 8080")
	log.Fatal(http.ListenAndServe(":8080", nil))
}

func hello(req *restful.Request, resp *restful.Response) {
	log.Println("request hello")
	_,_ = io.WriteString(resp, "world")
}
```

本地启动访问：

```bash
$ curl http://localhost:8080/hello
world
```

编译 ：

```bash
$ GOOS=linux GOARCH=amd64 go build
```



## 编写 envoy 的 yaml 配置文件

这里命名为 hello-envoy-config.yaml

```yaml
static_resources:
  listeners:
  - address:
      socket_address:
        address: 0.0.0.0
        port_value: 80
    filter_chains:
    - filters:
      - name: envoy.filters.network.http_connection_manager
        typed_config:
          "@type": type.googleapis.com/envoy.config.filter.network.http_connection_manager.v2.HttpConnectionManager
          codec_type: auto
          stat_prefix: ingress_http
          route_config:
            name: local_route
            virtual_hosts:
            - name: service
              domains:
              - "*"
              routes:
              - match:
                  prefix: "/hello"
                route:
                  cluster: local_service
          http_filters:
          - name: envoy.filters.http.router
            typed_config: {}
  clusters:
  - name: local_service
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    load_assignment:
      cluster_name: local_service
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: 127.0.0.1
                port_value: 8080
admin:
  access_log_path: "/dev/null"
  address:
    socket_address:
      address: 0.0.0.0
      port_value: 8081
```

这里说明几个关键点，`admin` 中配置了 envoy 的管理端口，`listeners` 中的 `address` 指定了 envoy 对外的服务地址和端口，`match` 表示匹配到之后转移到哪个 `cluster` ，`clusters` 中，指定了刚刚我们创建的服务端口。



## 创建启动脚本

创建名为 hello-start.sh 的文件：

```sh
#!/bin/sh
./code/hello-envoy &
envoy -c /etc/hello-envoy-config.yaml --service-cluster service
```





## 创建 Dockerfile 文件

创建名为 Dockerfile-hello 的文件：

```bash
FROM envoyproxy/envoy-alpine-dev:latest

RUN mkdir /code
ADD ./hello-envoy /code
ADD ./hello-start.sh /usr/local/bin/hello-start.sh
RUN chmod u+x /code/hello-envoy
RUN chmod u+x /usr/local/bin/hello-start.sh
ENTRYPOINT /usr/local/bin/hello-start.sh
```



## 创建 docker-compose 文件

创建名为 `docker-compose.yaml` 的文件：

```yaml
version: "3.7"
services:

  hello-service:
    build:
      context: .
      dockerfile: Dockerfile-hello
    volumes:
      - ./hello-envoy-config.yaml:/etc/hello-envoy-config.yaml
    networks:
      - envoymesh
    environment:
      - SERVICE_NAME=1
    expose:
      - "80"
      - "8081"
    ports:
      - "8000:80"
      - "8081:8081"

networks:
  envoymesh: {}
```

在这个 docker-compose 文件中，指定了 Dockerfile 文件，然后会把 `hello-envoy-config.yaml` 挂载到容器中，最后映射好端口。



## 上传 & 构建 & 测试

将上面的项目上传到服务器。然后进行构建：

```bash
$ sudo docker-compose up --build -d
```

测试：

```bash
$ sudo curl http://localhost:8000/hello
world
```

成功！！！

套路就是这么个套路，玩法就是这么个玩法。