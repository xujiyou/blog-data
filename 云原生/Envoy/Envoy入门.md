# Envoy 入门

为了深入学习 Istio，准备学习下 Envoy。Envoy 是 CNCF 中毕业的项目，具有很多高级特性。

Envoy：https://github.com/envoyproxy/envoy

参照教程：https://fuckcloudnative.io/posts/run-envoy-on-your-laptop/

---

## 安装

先安装 `docker-compose`

```bash
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.25.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

获取 envoy 的代码库：

```bash
$ git clone https://github.com/envoyproxy/envoy
$ cd envoy/examples/front-proxy
```

这个目录下有几个文件：

- service.py：定义了一个 Flask 后端服务
- service-envoy.yaml：定义了伴随 Flask 服务的 Envoy 服务
- Dockerfile-service：定义了在启动容器时同时启动 Flask 服务和其伴随的 Envoy 服务
- front-envoy.yaml：定义了前端代理的 Envoy 服务
- Dockerfile-frontenvoy：定义了前端代理的容器
- docker-compose.yaml：文件描述了如何构建、打包和运行前端代理与服务。

使用 docker-compose 启动容器，我这里将配置文件中的 8001 端口改成了 8002 端口，后面都是以 8002 为例：

```bash
$ docker-compose pull
$ docker-compose up --build -d
```

该命令将会启动一个前端代理和两个服务实例：service1 和 service2。

查看服务：

```bash
$ docker-compose ps
```



---



## 配置 Envoy

为了达到演示的目的，本文采用的是 Envoy 的静态配置。后续教程将会告诉你们如何使用动态配置来发挥 Envoy 的强大功能。

为了了解 Envoy 是如何配置的，先来看看 `docker-compose.yaml` 文件的前端代理部分的配置

```yaml
  front-envoy:
    build:
      context: .
      dockerfile: Dockerfile-frontenvoy
    volumes:
      - ./front-envoy.yaml:/etc/front-envoy.yaml
    networks:
      - envoymesh
    expose:
      - "80"
      - "8002"
    ports:
      - "8000:80"
      - "8002:8002"
```

解释一下，

- 使用 `Dockerfile-frontenvoy` 来构建镜像。
- 将 `front-envoy.yaml` 文件挂载为镜像中的 `/etc/front-envoy.yaml` 文件。
- 创建容器使用名为 `envoymesh` 的网络。
- 暴露 80 端口（用于一般通用流量）和 8001 端口（用于管理服务）。
- 将主机的 8000 端口和 8001 端口分别映射到容器的 80 端口和 8001 端口。



下面来看下 `front-envoy.yaml` ：

```yaml
  admin:
    access_log_path: "/dev/null"
    address:
      socket_address:
        address: 0.0.0.0
        port_value: 8002
```

admin 配置项的内容非常简单，`access_log_path` 字段的值设置为 `/dev/null`，意味着 admin 服务的访问日志将会被丢弃，在测试或生产环境中，你最好将这个值修改为不同的目录。socket_address 字段告诉 Envoy 创建一个监听在 8002 端口的 admin 服务。

`static_resources` 配置项定义了一组静态配置的集群（Cluster）和侦听器（Listener）。

**集群**是 Envoy 连接到的一组逻辑上相似的上游主机。Envoy 通过服务发现发现集群中的成员。Envoy 可以通过主动运行状况检查来确定集群成员的健康状况。Envoy 如何将请求路由到集群成员由负载均衡策略确定。

**侦听器**是服务(程序)监听者，就是真正干活的。 它是可以由下游客户端连接的命名网络位置（例如，端口、unix域套接字等）。



### Listener 配置

该示例中的前端代理有一个监听在 80 端口的侦听器，并配置了一个**监听器过滤器链**（filter_chains），用来管理 HTTP 流量：

在 HTTP 连接管理过滤器中，每一个虚拟主机都有单独的配置，并且都配置为接收所有域的流量：

```yaml
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
            - name: backend
              domains:
              - "*"
              routes:
              - match:
                  prefix: "/service/1"
                route:
                  cluster: service1
              - match:
                  prefix: "/service/2"
                route:
                  cluster: service2
          http_filters:
          - name: envoy.filters.http.router
            typed_config: {}
```

HTTP 路由规则将 `/service/1` 和 `/service/1` 的流量转发到各自的 Cluster。



### Cluster 配置

接下来看一下静态 Cluster 的定义：

```yaml
clusters:
  - name: service1
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    http2_protocol_options: {}
    load_assignment:
      cluster_name: service1
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: service1
                port_value: 80
  - name: service2
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    http2_protocol_options: {}
    load_assignment:
      cluster_name: service2
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: service2
                port_value: 80
```

在 Cluster 的配置中，你可以自定义超时、断路器和服务发现等。Cluster 由 Endpoint（端点）组成，其中 Endpoint 是一组可以为 Cluster 的请求提供服务的网络位置。本例中的 Endpoint 是通过 DNS 域名的方式定义的，Envoy 可以从域名中读取 Endpoint。Endpoint 也可以直接定义为 socket 地址，或者通过 `EDS`（Endpoint Discovery Service）动态读取。



## 访问服务

通过打开 `http://localhost:8000/service/1` 和 `http://localhost:8000/service/2` 来打开服务 。

另外 Envoy 的一大特色是内置的 Admin 服务，如果你在浏览器中访问 `http://localhost:8002` ，可以看到 Envoy admin 提供以下管理 API 端点。

通过 API 管理端可以对 Envoy 进行动态配置，参考 [v2 API reference](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api)。



---



## 卸载

```bash
$ docker-compose down
```



---



## 总结

通过上面的操作，知道了 Envoy 是怎么运行和配置的。后面还需要学 Envoy 是如何实现动态配置的，以及 Envoy 的 API 是怎样的，还有 Envoy 是如何配置来实现那些流量治理的功能的。











