# YARN 架构

官方文档：https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html

YARN的基本思想是将资源管理和作业调度/监视的功能拆分为单独的守护程序。想法是拥有一个全局ResourceManager（RM）和每个应用程序ApplicationMaster（AM）。应用程序可以是单个作业，也可以是作业的DAG。

ResourceManager 和 NodeManager 构成数据计算框架。 ResourceManager是在系统中所有应用程序之间仲裁资源的最终权限。 NodeManager是每台机器的框架代理，负责容器，监视其资源使用情况（cpu，内存，磁盘，网络），并将其报告给ResourceManager / Scheduler。

实际上，每个应用程序的ApplicationMaster是特定于框架的库，其任务是与来自ResourceManager的资源进行协商，并与NodeManager一起执行和监视任务。





