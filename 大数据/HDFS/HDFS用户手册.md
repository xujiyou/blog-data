# HDFS 用户手册

官方文档：https://hadoop.apache.org/docs/r3.3.0/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html

本文档是使用Hadoop分布式文件系统（HDFS）作为Hadoop集群的一部分或作为独立的通用分布式文件系统的用户的起点。尽管HDFS旨在在许多环境中“正常工作”，但HDFS的工作知识可极大地帮助改进特定群集上的配置和进行诊断。

## 概览

HDFS是Hadoop应用程序使用的主要分布式存储。 HDFS群集主要由管理文件系统元数据的NameNode和存储实际数据的DataNode组成。 《 HDFS体系结构指南》详细介绍了HDFS。本用户指南主要处理HDFS群集中用户和管理员的交互。 HDFS体系结构图描述了NameNode，DataNode和客户端之间的基本交互。客户端与NameNode联系以获取文件元数据或文件修改，并直接与DataNode执行实际的文件I / O。

以下是许多用户可能会感兴趣的一些重要功能：

- Hadoop（包括HDFS）非常适合使用商品硬件进行分布式存储和分布式处理。它具有容错性，可伸缩性，并且扩展极其简单。 MapReduce以其简单性和对大型分布式应用程序的适用性而闻名，它是Hadoop不可或缺的一部分。
- HDFS高度可配置，默认配置非常适合许多安装。在大多数情况下，仅需要为非常大的集群调整配置。
- Hadoop用Java编写，并且在所有主要平台上均受支持。
- Hadoop支持类 Shell 命令直接与HDFS进行交互。
- NameNode和Datanodes内置了Web服务器，可轻松检查群集的当前状态。
- HDFS会定期实施新功能和改进。以下是HDFS中有用功能的子集：
  - 文件权限和身份验证。
  - 机架感知：在计划任务和分配存储时考虑节点的物理位置。
  - 安全模式：一种维护的管理模式。
  - fsck：诊断文件系统运行状况，查找丢失的文件或块的实用程序。
  - fetchdt：一种实用程序，用于获取委托令牌并将其存储在本地系统上的文件中。
  - 平衡器：当数据在数据节点之间分布不均时，用于平衡集群的工具。
  - 升级和回滚：软件升级后，如果出现意外问题，可以在升级之前回滚到HDFS的状态。
  - Secondary NameNode：执行命名空间的定期检查点，并有助于将包含HDFS修改日志的文件的大小保持在NameNode的某些限制内。
  - Checkpoint节点：执行命名空间的定期检查点，并有助于最小化存储在NameNode上的日志大小，该日志包含对HDFS的更改。替换先前由次要NameNode填充的角色，尽管尚未经过战斗加固。只要没有在系统中注册任何备份节点，NameNode即可同时允许多个Checkpoint节点。
  - 备份节点：Checkpoint节点的扩展。除了检查点之外，它还从NameNode接收编辑流，并维护其自己的命名空间在内存中的副本，该副本始终与活动的NameNode命名空间状态保持同步。一次只能向NameNode注册一个备份节点。



## Web 接口

NameNode和DataNode各自运行一个内部Web服务器，以显示有关集群当前状态的基本信息。使用默认配置，NameNode主页位于 http:// namenode-name:9870 。它列出了集群中的DataNodes和集群的基本统计信息。 Web界面也可用于浏览文件系统（使用NameNode主页上的“浏览文件系统”链接）。



## Shell 命令

Hadoop包含各种类似于shell的命令，这些命令可直接与HDFS和Hadoop支持的其他文件系统进行交互。命令bin / hdfs dfs -help列出了Hadoop shell支持的命令。此外，命令`bin/hdfs dfs -help`命令名称显示命令的更多详细帮助。这些命令支持大多数常规文件系统操作，例如复制文件，更改文件权限等。它还支持一些HDFS特定操作，例如更改文件的复制。

#### DFSAdmin

`bin/hdfs dfsadmin`命令支持一些与HDFS管理相关的操作。 `bin/hdfs dfsadmin -help` 命令列出了当前支持的所有命令。例如：

- `-report`：报告HDFS的基本统计信息。这些信息中的某些信息也可以在NameNode主页上找到。
- `-safemode`：尽管通常不需要，但是管理员可以手动输入或退出安全模式。
- `-finalizeUpgrade`：删除上一次升级期间对群集所做的先前备份。
- `-refreshNodes`：使用允许连接到名称节点的数据节点集更新名称节点。默认情况下，Namenodes重读dfs.hosts，dfs.hosts.exclude定义的文件中的datanode主机名。dfs.hosts中定义的主机是属于群集的datanode。如果dfs.hosts中有条目，则只允许其中的主机向namenode注册。 dfs.hosts.exclude中的条目是需要停用的数据节点。或者，如果将dfs.namenode.hosts.provider.classname设置为org.apache.hadoop.hdfs.server.blockmanagement.CombinedHostFileManager，则所有包含和排除主机均在dfs.hosts定义的JSON文件中指定。当数据节点中的所有副本都复制到其他数据节点时，数据节点将完成退役。退役的节点不会自动关闭，也不会选择用于写入新副本。
- `-printTopology` : 打印集群的拓扑。显示由名称节点查看的连接到轨道的机架树和数据节点树。

比如 ：

```bash
$ sudo -u hdfs hdfs dfsadmin -printTopology
```



## Secondary NameNode

NameNode将对日志文件的修改存储为对日志文件的修改，并将日志存储在本地文件系统文件中。 NameNode启动时，它将从映像文件fsimage中读取HDFS状态，然后从编辑日志文件中应用编辑。然后，它将新的HDFS状态写入fsimage，并使用空的edits文件开始正常操作。由于NameNode仅在启动期间合并fsimage并编辑文件，因此在繁忙的群集上，随着时间的推移，编辑日志文件可能会变得非常大。较大的编辑文件的另一个副作用是，NameNode的下一次重新启动花费的时间更长。

辅助NameNode定期合并fsimage和edits日志文件，并将edits日志大小保持在限制范围内。它通常在与主要NameNode不同的机器上运行，因为其内存要求与主要NameNode的顺序相同。

辅助NameNode上检查点进程的开始由两个配置参数控制：

- `dfs.namenode.checkpoint.period` 默认设置为1小时，指定两个连续检查点之间的最大延迟。
- `dfs.namenode.checkpoint.txns` 默认设置为100万，它定义了NameNode上的非检查点事务数，即使未达到检查点期限，该事务也会强制执行紧急检查点。
- 次要NameNode将最新的检查点存储在目录中，该目录的结构与主要NameNode的目录相同。因此，如有必要，主NameNode始终可以准备好检查点图像。



## Checkpoint Node

NameNode使用两个文件来保留其名称空间：fsimage，它是名称空间的最新检查点，并进行编辑；自检查点以来，对名称空间所做的更改的日志（日志）。当NameNode启动时，它将合并fsimage并编辑日志以提供文件系统元数据的最新视图。然后，NameNode用新的HDFS状态覆盖fsimage并开始新的编辑日志。

Checkpoint节点定期创建名称空间的检查点。它从活动的NameNode下载fsimage并进行编辑，在本地进行合并，然后将新图像上传回活动的NameNode。 Checkpoint节点通常在与NameNode不同的机器上运行，因为其内存需求与NameNode的顺序相同。 Checkpoint节点由`bin/hdfs namenode -checkpoint`在配置文件中指定的节点上启动。

通过 `dfs.namenode.backup.address` 和 `dfs.namenode.backup.http-address` 配置变量配置Checkpoint（或Backup）节点及其随附的Web界面的位置。

Checkpoint节点上的checkpoint进程的启动由两个配置参数控制：

- `dfs.namenode.checkpoint.period` 默认设置为1小时，指定两个连续检查点之间的最大延迟
- `dfs.namenode.checkpoint.txns` 默认设置为100万，它定义了NameNode上的非检查点事务数，即使未达到检查点期限，该事务也会强制执行紧急检查点。

Checkpoint节点将最新的检查点存储在目录中，该目录的结构与NameNode的目录相同。如果需要的话，这可以使NameNode可以随时读取检查点图像。请参阅导入检查点。

可以在群集配置文件中指定多个检查点节点。



## Backup Node

备份节点提供与检查点节点相同的检查点功能，并维护始终与活动NameNode状态保持同步的文件系统名称空间的内存中最新副本。除了从NameNode接受文件系统编辑的日志流并将其持久保存到磁盘之外，Backup节点还将这些编辑应用到其在内存中的命名空间副本中，从而创建了命名空间的备份。

备份节点不需要从活动NameNode下载fsimage并编辑文件即可创建检查点，就像Checkpoint节点或Secondary NameNode所要求的那样，因为它已经具有名称空间状态的最新状态。备份节点检查点过程效率更高，因为它仅需要将名称空间保存到本地fsimage文件中并重置编辑。

NameNode一次支持一个备份节点。如果使用备份节点，则不能注册任何Checkpoint节点。将来将支持同时使用多个备份节点。

备份节点的配置方式与检查点节点相同。它从`bin/hdfs namenode -backup` 开始。

备份（或检查点）节点及其随附的Web界面的位置是通过 `dfs.namenode.backup.address` 和 `dfs.namenode.backup.http-address` 配置变量配置的。

使用备份节点提供了在没有持久性存储的情况下运行NameNode的选项，将将名称空间状态持久化的所有责任委托给了备份节点。为此，请使用-importCheckpoint选项启动NameNode，并为NameNode配置不指定类型为dfs.namenode.edits.dir的持久性存储目录。



## Import Checkpoint

如果图像的所有其他副本和编辑文件都丢失，则可以将最新的检查点导入到NameNode中。为了做到这一点，应该：

- 创建一个在 `dfs.namenode.name.dir` 配置变量中指定的空目录；
- 在配置变量 `dfs.namenode.checkpoint.dir`中指定检查点目录的位置。
- 并使用 `-importCheckpoint` 选项启动NameNode。

NameNode将从dfs.namenode.checkpoint.dir目录上载检查点，然后将其保存到dfs.namenode.name.dir中设置的NameNode目录中。如果dfs.namenode.name.dir中包含合法映像，则NameNode将失败。 NameNode验证dfs.namenode.checkpoint.dir中的映像是否一致，但不会以任何方式对其进行修改。



## Balancer

HDFS数据不一定总是在整个DataNode上均匀地放置。一个常见的原因是向现有群集中添加了新的DataNode。在放置新块（文件数据存储为一系列块）时，NameNode在选择DataNode接收这些块之前会考虑各种参数。一些注意事项是：

- 将块的副本之一与写入块的节点保留在同一节点上的策略。
- 需要在机架上散布块的不同副本，以便群集可以在整个机架丢失的情况下幸免。
- 复制副本之一通常与写入文件的节点放在同一机架上，从而减少了跨机架网络的I/O。
- 将HDFS数据均匀地分布在群集中的DataNode上。

出于多种竞争考虑，数据可能无法在整个DataNode上统一放置。 HDFS为管理员提供了一种工具，可以分析整个DataNode上的块放置和重新平衡数据。 HADOOP-1652提供了有关平衡器的简短管理员指南 [HADOOP-1652](https://issues.apache.org/jira/browse/HADOOP-1652)。

Balancer支持两种模式：作为工具或作为长期运行的服务运行：

- 在工具模式下，它将尝试尽最大努力平衡群集，并在以下情况下退出：
  - 所有集群都是平衡的。
  - 没有字节移动太多的迭代次数（默认为5）
  - 不能移动任何块。
  - 群集正在进行升级。
  - 其他错误

- 在服务模式下，balancer将作为长期运行的守护程序服务运行。它是这样的：
  - 对于每一轮，它将尝试平衡集群，直到成功或返回错误。
  - 您可以配置每轮之间的间隔，该间隔由 `dfs.balancer.service.interval` 设置。
  - 如果遇到意外的异常，它将在停止服务之前尝试几次，该服务由 `dfs.balancer.service.retries.on.exception` 设置。



## Rack Awareness

HDFS群集可以识别放置每个节点的机架的拓扑。配置此拓扑以优化数据容量和使用率很重要。有关更多详细信息，请检查通用文档中的机架识别。



## Safemode

在启动过程中，NameNode从fsimage和edits日志文件加载文件系统状态。然后，它等待DataNodes报告其块，以便尽管群集中已经存在足够的副本，它也不会过早开始复制这些块。在此期间，NameNode保持在安全模式。 NameNode的安全模式本质上是HDFS集群的只读模式，该模式不允许对文件系统或块进行任何修改。通常，在DataNode报告大多数文件系统块可用之后，NameNode会自动离开安全模式。如果需要，可以使用 `bin/hdfs dfsadmin -safemode` 命令将HDFS显式置于安全模式。 NameNode主页会显示安全模式是打开还是关闭。



## fsck

HDFS支持 `fsck` 命令来检查各种不一致情况。它设计用于报告各种文件的问题，例如，文件缺少的块或复制不足的块。与用于本机文件系统的传统 fsck 实用程序不同，此命令不会更正其检测到的错误。通常，NameNode会自动更正大多数可恢复的故障。默认情况下，fsck会忽略打开的文件，但提供在报告过程中选择所有文件的选项。 HDFS fsck命令不是Hadoop Shell命令。它可以作为`bin/hdfs fsck`运行。有关命令用法，请参见fsck。 fsck可以在整个文件系统或一部分文件上运行。

使用方式：

```bash
$ hdfs fsck /
```



## fetchdt

HDFS支持 `fetchdt` 命令来获取委托令牌并将其存储在本地系统上的文件中。以后可以使用此令牌从非安全客户端访问安全服务器（例如NameNode）。实用程序使用RPC或HTTPS（通过Kerberos）来获取令牌，因此需要在运行之前提供kerberos票证（运行kinit来获取票证）。 HDFS fetchdt命令不是Hadoop Shell命令。它可以作为`bin/hdfs fetchdt DTfile`运行。获得令牌后，可以通过将`HADOOP_TOKEN_FILE_LOCATION` 环境变量指向委托令牌文件来运行HDFS命令而无需Kerberos票证。



## Recovery Mode

通常，您将配置多个元数据存储位置。然后，如果一个存储位置损坏，则可以从其他存储位置之一读取元数据。

但是，如果仅有的可用存储位置已损坏，该怎么办？在这种情况下，有一种称为恢复模式的特殊NameNode启动模式，该模式可以允许您恢复大多数数据。

您可以像这样在恢复模式下启动NameNode：`namenode -recover`

在恢复模式下，NameNode将在命令行以交互方式提示您有关恢复数据可能采取的措施。

如果您不想被提示，可以给-force选项。此选项将强制恢复模式始终选择第一选项。通常，这将是最合理的选择。

由于恢复模式可能会导致数据丢失，因此在使用编辑日志和fsimage之前，应始终对其进行备份。



## 更新和恢复

当在现有集群上升级Hadoop以及进行任何软件升级时，可能存在影响现有应用程序的新错误或不兼容的更改，这些错误或不兼容的更改不会在早期发现。在任何不重要的HDFS安装中，都不可以丢失任何数据，更不用说从头开始重新启动HDFS了。 HDFS允许管理员返回到Hadoop的早期版本，并将群集回滚到升级之前的状态。 HDFS升级在Hadoop升级Wiki页面中有更详细的描述。 HDFS一次可以有一个这样的备份。升级之前，管理员需要使用 `bin/hadoop dfsadmin -finalizeUpgrade`命令删除现有备份。下面简要介绍典型的升级过程：

- 在升级Hadoop软件之前，请确定是否存在现有备份。
- 停止集群并分发新版本的Hadoop。
- 使用-upgrade选项运行新版本（sbin/start-dfs.sh -upgrade）。
- 大多数情况下，群集工作正常。一旦新的HDFS被认为运行良好（可能在运行几天后），请完成升级。请注意，在集群完成之前，删除升级之前存在的文件不会释放DataNode上的实际磁盘空间。
- 如果需要返回到旧版本：
  - 停止集群并分发早期版本的Hadoop。
  - 在namenode上运行rollback命令（bin/hdfs namenode -rollback）。
  - 使用回滚选项启动集群。 （sbin/start-dfs.sh -rollback）。

升级到新版本的HDFS时，必须重命名或删除新版本的HDFS中保留的所有路径。如果NameNode在升级过程中遇到保留路径，则将输出如下错误：

/.reserved is a reserved path and .snapshot is a reserved path component in this version of HDFS. Please rollback and delete or rename this path, or upgrade with the -renameReserved [key-value pairs] option to automatically rename these paths during upgrade.

指定 `-upgrade -renameReserved [可选键-值对]`会使NameNode自动重命名启动期间找到的所有保留路径。例如，要将所有名为.snapshot的路径重命名为.my-snapshot，并将.reserved重命名为.my-reserved，用户可以指定`-upgrade -renameReserved .snapshot=.my-snapshot，.reserved=.my-reserved`。

如果未使用-renameReserved指定键值对，则NameNode将使用。<LAYOUT-VERSION> .UPGRADE_RENAMED作为保留路径的后缀，例如.snapshot。-51.UPGRADE_RENAMED。

重命名过程有一些警告。如果可能的话，建议先升级`hdfs dfsadmin -saveNamespace`。这是因为，如果编辑日志操作引用自动重命名文件的目的地，则可能导致数据不一致。



## HDFS DataNode热插拔磁盘

Datanode 支持热插拔磁盘。可以添加或替换 HDFS 数据卷，而无需关闭 DataNode 。以下简要介绍了典型的热插拔磁盘过程：

- 如果有新的存储目录，则应格式化它们并挂载。
- 更新 DataNode 配置 `dfs.datanode.data.dir`加入或删掉数据卷目录。
- 运行 `hdfs dfsadmin -reconfig datanode HOST:PORT start`开始重新配置过程。可以使用 `hdfs dfsadmin -reconfig datanode HOST:PORT`状态来查询重新配置任务的运行状态。
- 重新配置任务完成后，用户可以安全地卸载已删除的数据卷目录并以物理方式删除磁盘。



## 文件权限和安全性

文件权限被设计为类似于其他熟悉的平台（如Linux）上的文件权限。当前，安全性仅限于简单文件权限。**启动NameNode的用户被视为HDFS的超级用户**。 HDFS的未来版本将支持诸如Kerberos之类的网络身份验证协议，用于用户身份验证和数据传输加密。有关详细信息，请参见“权限指南”。



## 可扩展性

Hadoop当前在具有数千个节点的集群上运行。 PoweredBy Wiki页面列出了一些在大型集群上部署Hadoop的组织。 HDFS每个群集都有一个NameNode。当前，NameNode上可用的总内存是主要的可伸缩性限制。在非常大的群集上，增加HDFS中存储的文件的平均大小有助于增加群集的大小，而不会增加NameNode上的内存要求。默认配置可能不适合非常大的群集。  [FAQ](http://wiki.apache.org/hadoop/FAQ) 页面列出了针对大型Hadoop集群的建议配置改进。





















