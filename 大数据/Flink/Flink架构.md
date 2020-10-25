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





## 任务和操作链

对于分布式执行，Flink将操作子任务链接到任务中。每个任务由一个线程执行。将操作链接到任务是一个有用的优化：它减少了线程到线程的切换和缓冲的开销，并增加了总体吞吐量，同时减少了延迟。可以配置链接行为。有关详细信息，请参见[chaining docs](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/dev/stream/operators/#task-chaining-and-resource-groups) 。

下图中的示例数据流由五个子任务执行，因此有五个并行线程。

![Operator chaining into Tasks](../../resource/tasks_chains.svg)



## 任务槽和资源

每个工作程序（TaskManager）是一个JVM进程，并且可以在单独的线程中执行一个或多个子任务。为了控制TaskManager接受多少个任务，它具有所谓的任务槽（至少一个）。

每个任务槽代表TaskManager的资源的固定子集。例如，具有三个插槽的TaskManager会将其托管内存的1/3专用于每个插槽。分配资源意味着子任务不会与其他作业的子任务竞争托管内存，而是具有一定数量的保留托管内存。请注意，此处没有发生CPU隔离。当前插槽仅将任务的托管内存分开。

通过调整任务槽的数量，用户可以定义子任务如何相互隔离。每个TaskManager具有一个插槽，这意味着每个任务组都在单独的JVM中运行（例如，可以在单独的容器中启动）。具有多个插槽意味着更多子任务共享同一JVM。同一JVM中的任务共享TCP连接（通过多路复用）和心跳消息。它们还可以共享数据集和数据结构，从而减少每个任务的开销。

![A TaskManager with Task Slots and Tasks](../../resource/tasks_slots.svg)



默认情况下，Flink允许子任务共享插槽，即使它们是不同任务的子任务也是如此，只要它们来自同一作业即可。结果是一个插槽可以容纳整个作业流水线。允许此插槽共享有两个主要好处：

- Flink集群所需的任务槽与作业中使用的最高并行度恰好一样。无需计算一个程序总共包含多少个任务（并行度各不相同）。
- 更容易获得更好的资源利用率。如果没有插槽共享，则非密集型source / map（）子任务将阻塞与资源密集型窗口子任务一样多的资源。通过插槽共享，在我们的示例中将基本并行度从2增加到6可充分利用插槽资源，同时确保沉重的子任务在TaskManager之间公平分配。

![TaskManagers with shared Task Slots](https://ci.apache.org/projects/flink/flink-docs-release-1.11/fig/slot_sharing.svg)





## Flink应用程序执行

Flink应用程序是从其 main() 方法中产生一个或多个 Flink 作业的任何用户程序。这些作业的执行可以在本地JVM（LocalEnvironment）或具有多台机器的集群的远程设置（RemoteEnvironment）中进行。对于每个程序，ExecutionEnvironment提供了一些方法来控制作业的执行（例如，设置并行性）并与外界进行交互（请参见[Anatomy of a Flink Program](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/dev/datastream_api.html#anatomy-of-a-flink-program)）。

Flink应用程序的作业可以提交到长时间运行的Flink会话群集，专用的Flink作业群集或Flink应用程序群集。这些选项之间的差异主要与集群的生命周期和资源隔离保证有关。



#### Flink Session Cluster

集群生命周期：在Flink会话群集中，客户端连接到一个可以长时间运行的群集，该群集可以接受多个作业提交。即使所有作业完成后，群集（和JobManager）也将继续运行，直到手动停止会话为止。因此，Flink会话群集的生存期不与任何Flink作业的生存期绑定。

资源隔离：TaskManager插槽由ResourceManager在作业提交时分配，并在作业完成后释放。因为所有作业都共享同一个群集，所以在群集资源方面存在一些竞争，例如提交作业阶段中的网络带宽。此共享设置的局限性在于，如果一个TaskManager崩溃，则所有在此TaskManager上运行任务的作业都将失败。以类似的方式，如果JobManager发生一些致命错误，它将影响集群中正在运行的所有作业。

其他注意事项：具有预先存在的群集可以节省大量时间来申请资源和启动TaskManager。在作业执行时间非常短且启动时间过长会对端到端用户体验产生负面影响的情况下（如对短查询进行交互分析的情况），这很重要，在这种情况下，希望作业可以快速完成。使用现有资源执行计算。

> 注意：以前，Flink会话群集在会话模式下也称为Flink群集。



#### Flink Job Cluster

集群生命周期：在Flink作业群集中，可用的群集管理器（例如YARN或Kubernetes）用于为每个提交的作业启动群集，并且该群集仅可用于该作业。在这里，客户端首先从群集管理器请求资源以启动JobManager，然后将作业提交给在此过程中运行的Dispatcher。然后根据作业的资源需求延迟分配TaskManager。作业完成后，Flink作业群集将被拆除。

资源隔离：JobManager中的致命错误仅影响在该Flink作业群集中运行的一个作业。

其他注意事项：因为ResourceManager必须应用并等待外部资源管理组件启动TaskManager进程并分配资源，所以Flink作业集群更适合于长时间运行，具有高稳定性要求且对较长启动时间不敏感的大型作业。

> 注意：以前，Flink作业群集在作业（或每个作业）模式下也称为Flink群集。

#### Flink Application Cluster

集群生命周期：Flink应用程序群集是专用的Flink群集，它仅从一个Flink应用程序执行作业，并且main（）方法在群集而不是客户端上运行。作业提交是一个单步过程：您无需先启动Flink群集，然后再将作业提交到现有的群集会话；相反，您将应用程序逻辑和依赖项打包到可执行作业JAR中，并且群集入口点（ApplicationClusterEntryPoint）负责调用main（）方法以提取JobGraph。例如，这使您可以像在Kubernetes上部署任何其他应用程序一样部署Flink应用程序。因此，Flink应用程序集群的生存期与Flink应用程序的生存期绑定在一起。

资源隔离：在Flink应用程序集群中，ResourceManager和Dispatcher的范围仅限于单个Flink应用程序，与Flink会话集群相比，它提供了更好的资源隔离。

> 注意：Flink作业集群可以看作是Flink应用程序集群的“客户端运行”替代方案。



























