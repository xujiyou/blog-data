# HiveServer2

官方文档：https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Overview

HiveServer2（HS2）是一项服务，使客户端能够对 Hive 执行查询。 HiveServer2 是已过时的 HiveServer1 的后继产品。 HS2 支持多客户端并发和身份验证。它旨在为 JDBC 和 ODBC 等开放 API 客户端提供更好的支持。

HS2 是作为复合服务运行的单个进程，其中包括基于 Thrift 的 Hive 服务（TCP或HTTP）和用于 Web UI 的 Jetty Web 服务器。



## 架构

基于 Thrift 的 Hive 服务是 HS2 的核心，并负责为 Hive 查询提供服务（例如，来自Beeline）。 Thrift 是用于构建跨平台服务的RPC框架。它的堆栈由4层组成：服务器，传输，协议和处理器。您可以在https://thrift.apache.org/docs/concepts中找到有关图层的更多详细信息。

HS2实现中这些层的用法如下所述。

#### Server

HS2将 TThreadPoolServer（来自Thrift）用于TCP模式，或将Jetty服务器用于HTTP模式。

TThreadPoolServer为每个TCP连接分配一个工作线程。每个线程始终与一个连接相关联，即使该连接处于空闲状态也是如此。因此，由于大量并发连接，大量线程导致了潜在的性能问题。将来，HS2可能会切换到另一种用于TCP模式的服务器类型，例如TThreadedSelectorServer。这是一篇有关不同Thrift Java服务器之间性能比较的[文章](https://github.com/m1ch1/mapkeeper/wiki/Thrift-Java-Servers-Compared)。

#### Transport

当客户端和服务器之间需要代理时（例如，出于负载平衡或安全原因），需要HTTP模式。这就是为什么支持它以及TCP模式的原因。您可以通过Hive配置属性hive.server2.transport.mode指定Thrift服务的传输模式。

#### Protocol

协议实现负责序列化和反序列化。 HS2目前正在使用 TBinaryProtocol 作为Thrift协议进行序列化。将来可能会基于更多性能评估来考虑其他协议，例如TCompactProtocol。

#### Processor

流程实现是处理请求的应用程序逻辑。例如，ThriftCLIService.ExecuteStatement（）方法实现了用于编译和执行Hive查询的逻辑。



## HS2 的依赖

- [Metastore](https://cwiki.apache.org/confluence/display/Hive/AdminManual+Metastore+Administration)

  可以将元存储库配置为嵌入式（与HS2相同的过程）或远程服务器（也基于Thrift的服务）。 HS2与元存储区对话以获取查询编译所需的元数据。

- Hadoop cluster

  HS2为各种执行引擎（MapReduce / Tez / Spark）准备物理执行计划，并将作业提交到Hadoop集群以执行。

可以在此处找到HS2及其依赖关系之间的[交互关系图](https://cwiki.apache.org/confluence/display/Hive/Design#Design-HiveArchitecture)。



## JDBC Client

建议客户端使用JDBC驱动程序与HS2进行交互。请注意，在某些用例（例如Hadoop Hue）中，直接使用Thrift客户端而绕过JDBC。

这是进行第一个查询所涉及的一系列API调用：

- JDBC客户端（例如Beeline）通过启动传输连接（例如TCP连接），然后发起OpenSession API调用来获取SessionHandle，从而创建HiveConnection 。会话是从服务器端创建的。
- 执行HiveStatement（遵循JDBC标准），并从Thrift客户端进行ExecuteStatement API调用。在API调用中，SessionHandle信息与查询信息一起传递到服务器。
- HS2服务器接收该请求，并请求驱动程序（它是CommandProcessor）进行查询解析和编译。驱动程序启动了一个将与Hadoop对话的后台作业，然后立即将响应返回给客户端。这是ExecuteStatement API的异步设计。响应包含从服务器端创建的OperationHandle。
- 客户端使用OperationHandle与HS2对话以轮询查询执行的状态。



























