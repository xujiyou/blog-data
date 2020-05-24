# HBase 配置概述

HBase 配置的官方文档：https://hbase.apache.org/book.html#configuration

Apache HBase使用与Apache Hadoop相同的配置系统。所有配置文件都位于conf /目录中，该文件需要与集群中的每个节点保持同步。

配置文件的种类：

- **backup-masters** 默认情况下不存在。一个纯文本文件，其中列出了主机应在其上启动备份主机进程的主机，每行一个主机。
- **hadoop-metrics2-hbase.properties** 用于连接HBase Hadoop的Metrics2框架。这是一个关于监控的文件。
- **hbase-env.cmd** 和 **hbase-env.sh** 用于设置HBase的环境变量，包括JAVA_HOME，Java选项和其他环境变量。该文件包含许多已注释掉的示例，以提供指导。
- **hbase-policy.xml** RPC服务器用来对客户端请求做出授权决策的默认策略配置文件。仅在启用HBase安全性时使用。
- **hbase-site.xml** 主要的 HBase 配置文件，该文件指定了配置选项，这些配置选项将覆盖HBase的默认配置。可以在HBase Web UI的  "HBase Configuration" 选项卡中查看群集的整个有效配置（默认和替代）。
- **log4j.properties** HBase 的 log4j 配置文件
- **regionservers** 一个纯文本文件，其中包含应在HBase群集中运行RegionServer的主机列表。默认情况下，此文件包含单个条目localhost。它应该包含一个主机名或IP地址的列表，每行一个，并且仅当群集中的每个节点都将在其localhost接口上运行RegionServer时才包含localhost。

