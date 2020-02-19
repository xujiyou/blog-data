# YARN 架构

YARN 的基本思想是将资源管理、作业调度、监控功能拆分成单独的守护进程。YARN 包括一个全局的 ResourceManager（RM）和若干个 ApplicationMaster（AM）组成，应用程序可以是单独的作业，也可以是作业的有向无环图。

ResourceManager 和 NodeManager 组成了数据计算框架。

ResourceManager 负责所有应用程序的权限。NodeManager 是每台机器的代理，负责监视容器的资源使用情况（cpu，内存，磁盘，网络），并报告给 ResourceManager。

每个 ApplicationMaster 包含一个任务，并且它与 ResourceManager 协商资源，然后与 NodeManager 一起执行和监视任务。

ResourceManager 包含两个组件：Scheduler和ApplicationsManager。

Scheduler 负责将资源分配给正在运行的应用程序，但要遵循容量队列的约束。调度器可插拔，有实时调度和公平调度。

ApplicationsManager 负责接收作业提交，并启动第一个 ApplicationMaster，并在失败时重启 ApplicationMaster。

ApplicationMaster 负责与调度程序协商适当的资源容器，跟踪其状态并监视进度。

