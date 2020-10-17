# YARN 架构

官方文档：https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html

YARN的基本思想是将资源管理和作业调度/监视的功能拆分为单独的守护程序。想法是拥有一个全局ResourceManager（RM）和每个应用程序ApplicationMaster（AM）。应用程序可以是单个作业，也可以是作业的DAG。

ResourceManager 和 NodeManager 构成数据计算框架。 ResourceManager是在系统中所有应用程序之间仲裁资源的最终权限。 NodeManager是每台机器的框架代理，负责容器，监视其资源使用情况（cpu，内存，磁盘，网络），并将其报告给ResourceManager / Scheduler。

实际上，每个应用程序的ApplicationMaster是特定于框架的库，其任务是与来自ResourceManager的资源进行协商，并与NodeManager一起执行和监视任务。

![MapReduce NextGen Architecture](../../resource/yarn_architecture.png)

ResourceManager具有两个主要组件：Scheduler 和 ApplicationsManager。

调度程序负责将资源分配给各种正在运行的应用程序，但要遵循熟悉的容量，队列等约束。调度程序在不对应用程序状态进行监视或跟踪的意义上是纯调度程序。此外，它也不保证由于应用程序故障或硬件故障而重新启动失败的任务。调度程序根据应用程序的资源需求执行调度功能；它基于资源容器的抽象概念来做到这一点，该容器包含诸如内存，cpu，磁盘，网络等元素。

调度程序具有可插拔策略，该策略负责在各种队列，应用程序等之间分配群集资源。当前的调度程序（例如CapacityScheduler和FairScheduler）将是一些插件示例。

ApplicationsManager负责接受作业提交，协商用于执行特定于应用程序的ApplicationMaster的第一个容器，并提供在失败时重新启动ApplicationMaster容器的服务。每个应用程序ApplicationMaster负责与调度程序协商适当的资源容器，跟踪其状态并监视进度。

hadoop-2.x中的MapReduce保持与以前的稳定版本（hadoop-1.x）的API兼容性。这意味着仅通过重新编译，所有MapReduce作业仍应在YARN上保持不变。

YARN通过ReservationSystem支持资源保留的概念，ReservationSystem是一个组件，该组件使用户可以指定资源随时间和时间限制（例如，截止日期）的配置文件，并保留资源以确保重要工作的可预测执行。ReservationSystem跟踪资源超时，执行保留的准入控制，并动态指示基础调度程序确保保留已满。

为了将YARN扩展到成千上万个节点，YARN通过YARN联合功能支持[YARN Federation](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/Federation.html)的概念。联合允许将多个纱线（子）簇透明地连接在一起，并使它们看起来像一个单一的簇。这可以用于实现更大的规模，和/或允许将多个独立的群集一起用于非常大的工作，或用于具有全部能力的租户。



















