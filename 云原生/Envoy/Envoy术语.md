# Envoy 术语

这里学习一下 Envoy 中的术语。

- **Upstream** 接收 Envoy 的连接和请求，并返回响应的服务端。 指的是 Envoy 背后的 Cluster 集群。
- **Downstream** 需要连接 Envoy，发送请求并接收响应。比如 curl 、浏览器、其他 Envoy 等。
- **Listener** 可以由下游客户端连接到的地址（例如端口，Unix域套接字等）。 Envoy 可以公开一个或多个 Listener 供下游客户端进行连接。
- **Cluster** 集群，由一组相同提供相同服务的程序组成，Envoy 通过服务发现来发现集群的成员。Envoy 可以主动通过健康检查来确定集群成员的运行状况。 Envoy 提供了多种负载均衡策略。
- **Mesh** 网格，一个 Envoy 和一组 Cluster 组成了一个网格，Envoy 守住了网格的入口和出口。
- **Runtime configuration** 运行期配置，Envoy 可以在运行期间实时更新配置。比如  xDS 服务端，RLS 服务端就可以在运行期修改配置并下发到 Envoy。