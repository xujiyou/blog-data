# Flink 架构

官方文档：https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/flink-architecture.html

Flink是一个分布式系统，需要有效分配和管理计算资源才能执行流应用程序。它与所有常见的群集资源管理器（如Hadoop YARN，Apache Mesos和Kubernetes）集成，但也可以设置为作为独立群集或库运行。

本节概述了Flink的体系结构，并描述了Flink的主要组件如何交互以执行应用程序并从故障中恢复。



## Flink群集剖析

Flink运行时由两种类型的进程组成：一个JobManager和一个或多个TaskManager。

![The processes involved in executing a Flink dataflow](../../resource/processes.svg)

客户端不是运行时和程序执行的一部分，而是用于准备数据流并将其发送到JobManager的。之后，客户端可以断开连接（分离模式），或保持连接状态以接收进度报告（连接模式）。客户端要么作为触发执行的 Java/Scala 程序的一部分运行，要么在命令行过程 ./bin/flink run ... 中运行。

JobManager 和 TaskManager 可以通过多种方式启动：直接作为独立群集在机器上，在容器中启动，或者由诸如 YARN 或 Mesos 的资源框架进行管理。 TaskManager 连接到 JobManager，宣布自己可用，并分配了工作。



#### JobManager

JobManager有许多与协调Flink应用程序的分布式执行相关的职责：它决定何时安排下一个任务（或一组任务），对完成的任务或执行失败做出反应，协调检查点并协调失败的恢复，其中包括其他。此过程包含三个不同的组件：

- **ResourceManager**

  ResourceManager负责Flink集群中的资源取消/分配和供应-它管理任务插槽，这些任务插槽是Flink集群中资源调度的单位（请参阅TaskManagers）。 Flink为不同的环境和资源提供者（例如YARN，Mesos，Kubernetes和独立部署）实现了多个ResourceManager。在独立设置中，ResourceManager只能分配可用TaskManager的插槽，而不能自行启动新的TaskManager。

- **Dispatcher**

  分派器提供REST API来提交Flink应用程序以供执行，并为每个提交的作业启动一个新的JobMaster。它还运行Flink WebUI以提供有关作业执行的信息。

- **JobMaster**

  JobMaster负责管理单个JobGraph的执行。 Flink群集中可以同时运行多个作业，每个作业都有自己的JobMaster。

始终至少有一个JobManager。高可用性设置可能有多个JobManager，其中一个始终是领导者，而其他则处于待机状态（请参阅[High Availability (HA)](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/ops/jobmanager_high_availability.html)）。



#### TaskManagers

TaskManager（也称为工作程序）执行数据流的任务，并缓冲和交换数据流。

必须始终至少有一个TaskManager。 TaskManager中资源调度的最小单位是任务槽。 TaskManager中任务插槽的数量指示并发处理任务的数量。请注意，多个操作可以在一个任务槽中执行（请参阅 [Tasks and Operator Chains](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/flink-architecture.html#tasks-and-operator-chains)）。



































