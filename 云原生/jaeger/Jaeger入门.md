# Jaeger 入门

安装 Istio 后，执行：

```
$ kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=jaeger -o jsonpath='{.items[0].metadata.name}') 15032:16686 &
```

然后就可以通过 `http://localhost:15032`来访问jaeger的UI



不依赖 Istio，下面单独装一个玩玩。

首先下载 jaeger 的二进制文件，下载地址：https://github.com/jaegertracing/jaeger/releases

启动：

```bash
$ jaeger-all-in-one --collector.zipkin.http-port=9411
```

UI 界面在 http://localhost:16686 



## 架构

先来学习下术语。

### Span

一个 Span 是一个逻辑工作单元，其具有操作名称，开始时间和持续时间。 Span 可以嵌套，从而建立因果关系模型。

### Trace

Trace 是一个系统的 数据路径或执行路径，并且可以作为一个有向无环图。



## 组件

Jaeger 可以部署为一个 one in all 二进制程序，也可以作为一个可扩展的分布式系统。

Jaeger 可以直接写入本地储存，也可以将数据传输给 kafka 。

*直接存储架构插图*：![architecture-v1](/Users/jiyouxu/Documents/me/blog/resource/architecture-v1.png)

*以Kafka作为中间缓冲区的体系结构插图*：

![建筑](/Users/jiyouxu/Documents/me/blog/resource/architecture-v2.png)



### Jaeger客户端库

Jaeger客户端是[OpenTracing API的](https://opentracing.io/)特定于语言的实现。它们可用于手动或通过与OpenTracing集成的各种现有开放源代码框架（例如Flask，Dropwizard，gRPC等）来检测应用程序以进行分布式跟踪。

### 代理

Jaeger **代理**是一个网络守护程序，它侦听通过UDP发送的跨度，并将其分批发送给收集器。它旨在作为基础结构组件部署到所有主机。该代理将收集器的路由和发现抽象为远离客户端。

### Collector

负责从Jaeger [代理](https://www.jaegertracing.io/docs/1.17/architecture#agent)接收 Trace 数据，并通过处理管道运行它们。当前，我们的管道会验证 Trace，为其建立索引，执行任何转换并最终存储它们。

Jaeger的存储是一个可插拔组件，目前支持[Cassandra](https://www.jaegertracing.io/docs/1.17/deployment#cassandra)，[Elasticsearch](https://www.jaegertracing.io/docs/1.17/deployment#elasticsearch)和[Kafka](https://www.jaegertracing.io/docs/1.17/deployment#kafka)。

### Query

Query 是一项从存储中检索 Tracee 并托管UI来显示跟踪的服务。

### Ingester

**Ingester**是一项从 Kafka Topic 读取并写入另一个存储后端（Cassandra，Elasticsearch）的服务。



## API

Jaeger组件实现了各种API，用于保存或检索跟踪数据。

API 接收几种数据格式

保存 API 数据格式

- Thrift over UDP (stable)

- Protobuf via gRPC (stable）

- Thrift over HTTP (stable)

检索 API 数据格式

- gRPC/Protobuf (stable)

- HTTP JSON (internal)

- Clients configuration (internal)

- Service dependencies graph (internal)









