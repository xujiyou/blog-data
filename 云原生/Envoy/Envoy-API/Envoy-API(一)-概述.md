# Envoy API （一） -  概述

官方 API 文档：https://www.envoyproxy.io/docs/envoy/latest/api/api

目前（Envoy 版本1.15.0）API 版本分为 v2 和 v3。我这里主要学习一下 v2 版本的 API。。。。

传输版本和资源版本可能混合在一起。例如，可以通过v2传输协议来传输v3资源。此外，Envoy 可能会为不同的资源类型使用混合的资源版本。例如，v3 Cluster 可与 v2 Listener 一起使用。

xDS 的 API 可以通过 REST 访问，也可以通过 gRPC 访问，REST 访问简单直观，适合调试，gRPC 访问性能好，适合用于正式环境。

注意，在后面会将 `API` 和 `配置` 这两个术语混用，因为他俩归根结底是一回事。



## v2 API reference

v2 版本的 API 参考：https://www.envoyproxy.io/docs/envoy/latest/api-v2/api

共分为 10 种类型。分别是：

- Boorstrap ：必须要配置的，不管是动态配置还是静态配置，这个类型包含其他类型，直接包含 SDS、RTDS 相关 API
- Listeners：LDS 相关API
- Clusters：CDS 相关API
- HTTP Route Manager：包含 RDS、VHDS、SRDS 相关 API
- Extensions
- Admin：Envoy 的管理端配置
- Envoy Data
- Services
- Common messages
- Tpyes

后面的文章分别对这些 API 进行动手学习。