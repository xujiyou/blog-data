# 使用 QJM 实现 HDFS 高可用

本指南概述了HDFS高可用性（HA）功能以及如何使用Quorum Journal Manager（QJM）功能配置和管理HA HDFS群集。

本文档假定读者对HDFS群集中的常规组件和节点类型有一般的了解。有关详细信息，请参阅HDFS体系结构指南。



## 背景

在Hadoop 2.0.0之前，NameNode是HDFS集群中的单点故障（SPOF）。每个群集只有一个NameNode，并且如果该计算机或进程不可用，则整个群集将不可用，直到NameNode重新启动或在单独的计算机上启动。

这从两个方面影响了HDFS群集的总可用性：

- 在发生意外事件（例如机器崩溃）的情况下，群集将不可用，直到操作员重新启动NameNode。
- 计划的维护事件，例如NameNode计算机上的软件或硬件升级，将导致群集停机时间的延长。

HDFS高可用性功能通过提供以下选项来解决上述问题：在具有热备用的主动/被动配置中，可以在同一群集中运行两个（并且自3.0.0起，超过两个）冗余NameNode。这可以在计算机崩溃的情况下快速故障转移到新的NameNode，或者出于计划维护的目的由管理员发起的正常故障转移。



## 架构

在典型的HA群集中，将两个或更多单独的计算机配置为NameNode。在任何时间点，恰好其中一个NameNode处于Active状态，而其他NameNode处于Standby状态。 Active NameNode负责群集中的所有客户端操作，而Standby则仅充当工作程序，并保持足够的状态以在必要时提供快速故障转移。

为了使备用节点保持其状态与活动节点同步，两个节点都与一组称为“ JournalNodes”（JN）的单独守护程序进行通信。当活动节点执行任何名称空间修改时，它会持久地将修改记录记录到这些JN的大多数中。 Standby节点能够从JN中读取编辑，并一直在监视它们以查看编辑日志的更改。当“备用节点”看到编辑内容时，会将其应用到自己的名称空间。发生故障转移时，备用服务器将确保在将自身提升为活动状态之前，已从JournalNode读取所有编辑内容。这样可确保在发生故障转移之前，名称空间状态已完全同步。

为了提供快速故障转移，备用节点还必须具有有关集群中块位置的最新信息。为了实现这一点，DataNode被配置了所有NameNode的位置，并向所有人发送块位置信息和心跳。

对于HA群集的正确操作至关重要，一次只能有一个NameNode处于活动状态。否则，名称空间状态将在两者之间迅速分散，从而有数据丢失或其他不正确结果的风险。为了确保此属性并防止所谓的“裂脑情况”，JournalNode将仅一次允许单个NameNode成为作者。在故障转移期间，将变为活动状态的NameNode将仅承担写入JournalNodes的角色，这将有效地防止另一个NameNode继续处于活动状态，从而使新的Active节点可以安全地进行故障转移。



## 硬件资源

为了部署HA群集，您应该准备以下内容：

- **NameNode machines** 运行主节点和备用名称节点的计算机应具有彼此等效的硬件，并且应具有与非HA群集中使用的硬件等效的硬件。
- **JournalNode machines** 运行JournalNodes的计算机。 JournalNode守护程序相对较轻，因此可以合理地将这些守护程序与其他Hadoop守护程序（例如NameNode，JobTracker或YARN ResourceManager）并置在计算机上。注意：必须至少有3个JournalNode守护程序，因为必须将编辑日志修改写入大多数JN。这将允许系统容忍单个计算机的故障。您可能还会运行3个以上的JournalNode，但是为了实际增加系统可以容忍的故障数量，您应该运行奇数个JN（即3、5、7等）。请注意，当与N个JournalNode一起运行时，系统最多可以容忍（N-1）/ 2个故障，并继续正常运行。

请注意，在高可用性群集中，备用名称节点也执行名称空间状态的检查点，因此不必在高可用性群集中运行辅助名称节点，检查点节点或备份节点。实际上，这样做将是一个错误。这还允许正在重新配置未启用HA的HDFS群集的用户启用HA，以重用他们先前专用于次要NameNode的硬件。



## 配置

与联合身份验证配置类似，高可用性配置向后兼容，并允许现有的单个NameNode配置无需更改即可工作。设计新配置后，群集中的所有节点都可以具有相同的配置，而无需根据节点的类型将不同的配置文件部署到不同的机器上。

与HDFS联合身份验证一样，HA群集重用名称服务ID来标识单个HDFS实例，该实例实际上可能包含多个HA NameNode。此外，HA还添加了一个名为NameNode ID的新抽象。群集中的每个不同的NameNode都有一个不同的NameNode ID来区分它。为了支持所有NameNode的单个配置文件，相关的配置参数后缀有nameservice ID和NameNode ID。



要配置HA NameNode，必须将多个配置选项添加到hdfs-site.xml配置文件中。

设置这些配置的顺序并不重要，但是您为dfs.nameservices和dfs.ha.namenodes。[nameservice ID]选择的值将确定后面的密钥。因此，您应该在设置其余配置选项之前决定这些值。

- **dfs.nameservices** 新名称服务的逻辑名称

  选择此名称服务的逻辑名称，例如“ mycluster”，然后将此逻辑名称用作此配置选项的值。您选择的名称是任意的。它既可以用于配置，也可以用作集群中绝对HDFS路径的权限组件。

  注意：如果您还使用HDFS Federation，则此配置设置还应包括其他名称服务（HA或其他）的列表，以逗号分隔的列表。

  ```xml
  <property>
    <name>dfs.nameservices</name>
    <value>mycluster</value>
  </property>
  ```

- **dfs.ha.namenodes.[nameservice ID]** 名称服务中每个NameNode的唯一标识符

  使用逗号分隔的NameNode ID列表进行配置。 DataNode将使用它来确定集群中的所有NameNode。例如，如果您以前使用“ mycluster”作为名称服务ID，并且想要使用“ nn1”，“ nn2”和“ nn3”作为NameNode的各个ID，则可以这样配置：

  ```xml
  <property>
    <name>dfs.ha.namenodes.mycluster</name>
    <value>nn1,nn2, nn3</value>
  </property>
  ```

  注意：用于HA的NameNode的最小数量为2，但是您可以配置更多。由于通信开销，建议不要超过5个（建议使用3个NameNode）。

- **dfs.namenode.rpc-address.[nameservice ID].[name node ID]** 每个NameNode监听的标准RPC地址

  对于两个先前配置的NameNode ID，请设置NameNode进程的完整地址和IPC端口。请注意，这将导致两个单独的配置选项。例如：

  ```xml
  <property>
    <name>dfs.namenode.rpc-address.mycluster.nn1</name>
    <value>machine1.example.com:8020</value>
  </property>
  <property>
    <name>dfs.namenode.rpc-address.mycluster.nn2</name>
    <value>machine2.example.com:8020</value>
  </property>
  <property>
    <name>dfs.namenode.rpc-address.mycluster.nn3</name>
    <value>machine3.example.com:8020</value>
  </property>
  ```

  注意：如果需要，您可以类似地配置“ servicerpc-address”设置。

- **dfs.namenode.http-address.[nameservice ID].[name node ID]** 每个NameNode监听的标准HTTP地址

  与上面的rpc-address类似，为两个NameNode的HTTP服务器设置地址以进行侦听。例如：

  ```xml
  <property>
    <name>dfs.namenode.http-address.mycluster.nn1</name>
    <value>machine1.example.com:9870</value>
  </property>
  <property>
    <name>dfs.namenode.http-address.mycluster.nn2</name>
    <value>machine2.example.com:9870</value>
  </property>
  <property>
    <name>dfs.namenode.http-address.mycluster.nn3</name>
    <value>machine3.example.com:9870</value>
  </property>
  ```

- **dfs.namenode.shared.edits.dir** URI，它标识NameNode将在其中写入/读取编辑内容的JN组

  在这里，可以配置提供共享编辑存储的JournalNode的地址，该地址由Active nameNode写入并由Standby NameNode读取，以与Active NameNode所做的所有文件系统更改保持最新。尽管您必须指定多个JournalNode地址，但是您仅应配置这些URI之一。 URI的格式应为：qjournal://host1:port1;host2:port2;host3:port3/journalId。日记ID是此名称服务的唯一标识符，它允许单个JournalNode集为多个联合名称系统提供存储。尽管不是必需的，但最好将名称服务ID用作日记标识符。

  例如，如果此群集的JournalNode在计算机“ node1.example.com”，“ node2.example.com”和“ node3.example.com”上运行，并且名称服务ID为“ mycluster”，则应使用以下为该设置的值（JournalNode的默认端口为8485）：

  ```xml
  <property>
    <name>dfs.namenode.shared.edits.dir</name>
    <value>qjournal://node1.example.com:8485;node2.example.com:8485;node3.example.com:8485/mycluster</value>
  </property>
  ```

- **dfs.client.failover.proxy.provider.[nameservice ID]** HDFS客户端用于联系Active NameNode的Java类

  配置Java类的名称，DFS客户端将使用该Java类来确定哪个NameNode是当前的Active，从而确定哪个NameNode当前正在服务于客户端请求。 Hadoop当前附带的两个实现是ConfiguredFailoverProxyProvider和RequestHedgingProxyProvider（对于第一个调用，它们同时调用所有名称节点以确定活动的名称节点，并在后续请求时调用活动的名称节点，直到发生故障转移），因此除非您使用自定义代理提供程序，否则请使用其中之一。例如：

  ```xml
  <property>
    <name>dfs.client.failover.proxy.provider.mycluster</name>
    <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
  </property>
  ```

- **dfs.ha.fencing.methods** 故障转移期间将用于隔离Active NameNode的脚本或Java类的列表

  为了保证系统的正确性，在任何给定时间只有一个NameNode处于Active状态。重要的是，当使用Quorum Journal Manager时，将只允许一个NameNode写入JournalNodes，因此不会因裂脑情况而损坏文件系统元数据。但是，当发生故障转移时，以前的Active NameNode仍可能会向客户端提供读取请求，这可能已过期，直到该NameNode在尝试写入JournalNodes时关闭为止。因此，即使使用Quorum Journal Manager，仍然需要配置一些防护方法。但是，为了在防护机制失败的情况下提高系统的可用性，建议配置一种防护方法，以确保成功返回列表中的最后一种防护方法。请注意，如果您选择不使用任何实际的防护方法，则仍必须为此设置配置一些内容，例如“ shell（/ bin / true）”。

  在故障转移期间使用的防护方法配置为以回车符分隔的列表，将按顺序尝试该列表，直到指示防护成功为止。 Hadoop附带两种方法：shell和sshfence。有关实现自己的自定义防护方法的信息，请参见org.apache.hadoop.ha.NodeFencer类。

  **sshfence** SSH到Active NameNode并终止进程

  sshfence选项通过SSH连接到目标节点，并使用熔凝器杀死监听该服务的TCP端口的进程。为了使该防护选项起作用，它必须能够在不提供密码的情况下SSH到目标节点。因此，还必须配置dfs.ha.fencing.ssh.private-key-files选项，该选项是用逗号分隔的SSH私钥文件列表。例如：

  ```xml
      <property>
        <name>dfs.ha.fencing.methods</name>
        <value>sshfence</value>
      </property>
  
      <property>
        <name>dfs.ha.fencing.ssh.private-key-files</name>
        <value>/home/exampleuser/.ssh/id_rsa</value>
      </property>
  ```

  可以选择配置一个非标准的用户名或端口来执行SSH。还可以为SSH配置一个以毫秒为单位的超时，此后该防护方法将被视为失败。可以这样配置：

  ```xml
      <property>
        <name>dfs.ha.fencing.methods</name>
        <value>sshfence([[username][:port]])</value>
      </property>
      <property>
        <name>dfs.ha.fencing.ssh.connect-timeout</name>
        <value>30000</value>
      </property>
  ```

  **shell** 运行任意的shell命令以隔离Active NameNode

  shell防护方法运行一个任意的shell命令。可以这样配置：

  ```xml
      <property>
        <name>dfs.ha.fencing.methods</name>
        <value>shell(/path/to/my/script.sh arg1 arg2 ...)</value>
      </property>
  ```

  括号之间的字符串将直接传递到bash shell，并且可能不包含任何右括号。

  shell命令将在一个设置为包含所有当前Hadoop配置变量的环境中运行，并用“ _”字符替换配置键中的任何“。”字符。所使用的配置已经将任何特定于名称节点的配置提升为通用形式，例如dfs_namenode_rpc-address将包含目标节点的RPC地址，即使该配置可以将该变量指定为dfs.namenode.rpc-address.ns1 .nn1。

  此外，还提供以下变量，这些变量引用要隔离的目标节点：

  |                       |                                           |
  | :-------------------- | ----------------------------------------- |
  | $target_host          | hostname of the node to be fenced         |
  | $target_port          | IPC port of the node to be fenced         |
  | $target_address       | the above two, combined as host:port      |
  | $target_nameserviceid | the nameservice ID of the NN to be fenced |
  | $target_namenodeid    | the namenode ID of the NN to be fenced    |

  这些环境变量也可以在shell命令本身中用作替代。例如：

  ```xml
      <property>
        <name>dfs.ha.fencing.methods</name>
        <value>shell(/path/to/my/script.sh --nameservice=$target_nameserviceid $target_host:$target_port)</value>
      </property>
  ```

  如果shell命令返回退出代码0，则确定防护成功。如果返回任何其他退出代码，则防护不成功，并且将尝试列表中的下一个防护方法。

  注意：此防护方法不会实现任何超时。如果需要超时，则应在shell脚本本身中实现超时（例如，通过分叉subshell在几秒钟内杀死其父级）。

- **fs.defaultFS** 没有给出Hadoop FS客户端使用的默认路径前缀

  可选，可以配置Hadoop客户端的默认路径，以使用新的启用HA的逻辑URI。如果您之前使用“ mycluster”作为名称服务ID，则它将是所有HDFS路径的授权部分的值。可以这样配置，在您的core-site.xml文件中：

  ```xml
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://mycluster</value>
  </property>
  ```

- **dfs.journalnode.edits.dir** JournalNode守护程序将存储其本地状态的路径

  这是JournalNode机器上将存储JN使用的编辑和其他本地状态的绝对路径。您只能为此配置使用一条路径。通过运行多个单独的JournalNode或在本地连接的RAID阵列上配置此目录，可以提供此数据的冗余。例如：

  ```xml
  <property>
    <name>dfs.journalnode.edits.dir</name>
    <value>/path/to/journal/node/local/data</value>
  </property>
  ```

- **dfs.ha.nn.not-become-active-in-safemode** 如果阻止安全模式名称节点变为活动状态

  当自动故障转移打开时，是否允许namenode在安全模式下变为活动状态，将其设置为true时，安全模式下的namenode将向ZKFC报告SERVICE_UNHEALTHY，或者在自动故障转移关闭的情况下引发异常，从而无法转换为活动状态。例如：

  ```xml
  <property>
    <name>dfs.ha.nn.not-become-active-in-safemode</name>
    <value>true</value>
  </property>
  ```



## 部署

设置完所有必需的配置选项后，必须在将要运行它们的机器集上启动JournalNode守护程序。这可以通过运行命令`hdfs --daemon start journalnode`并等待该守护程序在每台相关计算机上启动来完成。

一旦启动JournalNode，便必须首先同步两个HA NameNode的磁盘元数据。

- 如果要设置新的HDFS群集，则应首先在其中一个NameNode上运行format命令（hdfs namenode -format）。
- 如果您已经格式化了NameNode，或者正在将未启用HA的群集转换为启用HA，则现在应该通过运行以下命令将NameNode元数据目录的内容复制到其他未格式化的NameNode：未经格式化的NameNode上的`hdfs namenode -bootstrapStandby`。运行此命令还将确保JournalNode（由dfs.namenode.shared.edits.dir配置）包含足够的编辑事务以能够启动两个NameNode。
- 如果要将非HA NameNode转换为HA，则应运行命令`hdfs namenode -initializeSharedEdits`，该命令将使用本地NameNode edits目录中的edits数据初始化JournalNode。

此时，您可以像通常启动NameNode一样启动所有HA NameNode。

通过浏览到每个NameNode配置的HTTP地址来分别访问它们。您应注意，配置的地址旁边将是NameNode的HA状态（“待机”或“活动”）。无论何时启动HA NameNode，它最初都处于Standby状态。



## 管理命令

现在您的HA NameNodes已配置并启动，您将可以访问一些其他命令来管理HA HDFS群集。具体来说，您应该熟悉“ hdfs haadmin”命令的所有子命令。在不使用任何其他参数的情况下运行此命令将显示以下用法信息：

```
Usage: haadmin
    [-transitionToActive <serviceId>]
    [-transitionToStandby <serviceId>]
    [-failover [--forcefence] [--forceactive] <serviceId> <serviceId>]
    [-getServiceState <serviceId>]
    [-getAllServiceState]
    [-checkHealth <serviceId>]
    [-help <command>]
```

本指南描述了每个子命令的高级用法。有关每个子命令的特定用法信息，应运行“ hdfs haadmin -help <命令>”。

- **transitionToActive** and **transitionToStandby** 将给定NameNode的状态转换为Active或Standby

  这些子命令使给定的NameNode分别转换为Active或Standby状态。这些命令不会尝试执行任何防护，因此应很少使用。相反，几乎应该总是喜欢使用“ hdfs haadmin -failover”子命令。

- **failover** 在两个NameNode之间启动故障转移

  此子命令导致从第一个提供的NameNode到第二个的NameNode故障转移。如果第一个NameNode处于Standby状态，此命令将简单地将第二个NameNode转换为Active状态，而不会出现错误。如果第一个NameNode处于活动状态，则将尝试将其优雅地转换为Standby状态。如果失败，将按顺序尝试防护方法（由dfs.ha.fencing.methods配置），直到成功为止。仅在此过程之后，第二个NameNode才会转换为Active状态。如果没有成功的防护方法，则第二个NameNode将不会转换为Active状态，并且将返回错误。

- **getServiceState** 确定给定的NameNode是活动的还是备用的

  连接到提供的 NameNode 以确定其当前状态，并在STDOUT上适当打印“待机”或“活动”。 cron作业或监视脚本可能会使用此子命令，这些脚本或监视脚本需要根据NameNode当前处于活动状态还是待机状态而表现不同。

- **getAllServiceState** 返回所有NameNode的状态

  连接到已配置的NameNode以确定当前状态，并在STDOUT上适当打印“待机”或“活动”。

- **checkHealth** 检查给定NameNode的运行状况

  连接到提供的NameNode以检查其运行状况。 NameNode能够对其自身执行一些诊断，包括检查内部服务是否按预期运行。如果NameNode正常，此命令将返回0，否则返回非零。人们可能会使用此命令进行监视。

  注意：这尚未实现，除非给定的NameNode完全关闭，否则当前将始终返回成功。



## 负载均衡

如果您正在负载平衡器（例如Azure或AWS）后面运行一组NameNode，并且希望负载平衡器指向活动的NN，则可以将 /isActive HTTP 端点用作运行状况探测器。如果NN处于活动HA状态，则http://NN_HOSTNAME/isActive将返回200状态代码响应，否则返回405。



## 进行中的编辑日志拖尾

在默认设置下，Standby NameNode将仅应用已完成的编辑日志段中存在的编辑。如果希望具有一个具有最新名称空间信息的Standby NameNode，则可以启用正在进行的编辑段的尾部处理。此设置将尝试从JournalNode上的内存高速缓存中获取编辑，并且可以将在Standby NameNode上应用事务之前的延迟时间减少到毫秒量级。如果无法从缓存中进行编辑，则备用数据库仍将能够检索它，但是滞后时间将更长。相关配置为：

- **dfs.ha.tail-edits.in-progress** 是否在进行中的编辑日志中启用拖尾。这还将在JournalNodes上启用内存中编辑缓存。默认禁用。
- **dfs.journalnode.edit-cache-size.bytes** JournalNode上的内存中编辑缓存的大小。在典型的环境中，每个编辑占用大约200个字节，因此，例如，默认值1048576（1MB）可以容纳大约5000个事务。建议监视JournalNode指标RpcRequestCacheMissAmountNumMisses和RpcRequestCacheMissAmountAvgTxns，它们分别计算无法由缓存服务的请求数，以及为成功请求而必须存在于缓存中的额外事务数。例如，如果请求尝试从事务ID 10开始获取编辑，但是缓存中最旧的数据在事务ID 20，则将平均值添加10。

该功能主要与“待机/观察者读取”功能结合使用。使用此功能，可以从非活动的NameNode服务读取请求；因此，进行中的后期编辑为这些节点提供了为请求提供最新数据的能力。有关此功能的更多信息，请参见Apache JIRA票证HDFS-12943。



## 自动故障转移

以上各节描述了如何配置手动故障转移。在这种模式下，即使活动节点发生故障，系统也不会自动触发从活动NameNode到备用NameNode的故障转移。本节介绍如何配置和部署自动故障转移。

#### 组件

自动故障转移为HDFS部署添加了两个新组件：ZooKeeper 和 ZKFailoverController进程（缩写为ZKFC）。

Apache ZooKeeper是一项高可用性服务，用于维护少量的协调数据，将数据中的更改通知客户端并监视客户端的故障。 HDFS自动故障转移的实现依赖ZooKeeper进行以下操作：

- **Failure detection** 群集中的每个NameNode计算机都在ZooKeeper中维护一个持久性会话。如果计算机崩溃，则ZooKeeper会话将过期，通知其他NameNode应该触发故障转移。
- **Active NameNode election** ZooKeeper提供了一种简单的机制来专门选择一个节点为活动节点。如果当前活动的NameNode崩溃，则另一个节点可能会在ZooKeeper中采取特殊的排他锁，指示它应该成为下一个活动的NameNode。

ZKFailoverController（ZKFC）是一个新组件，它是一个ZooKeeper客户端，它还监视和管理NameNode的状态。运行NameNode的每台计算机也都运行ZKFC，该ZKFC负责：

- **Health monitoring** ZKFC使用运行状况检查命令定期ping其本地NameNode。只要NameNode以健康状态及时响应，ZKFC就会认为该节点健康。如果节点崩溃，冻结或以其他方式进入不正常状态，则运行状况监视器将其标记为不正常。
- **ZooKeeper session management** 当本地NameNode运行状况良好时，ZKFC会在ZooKeeper中保持一个打开的会话。如果本地NameNode处于活动状态，则它还将持有一个特殊的“锁定” znode。此锁使用ZooKeeper对“临时”节点的支持；如果会话期满，将自动删除锁定节点。
- **ZooKeeper-based election** 如果本地NameNode运行状况良好，并且ZKFC看到当前没有其他节点持有锁znode，则它本身将尝试获取该锁。如果成功，则表明它“赢得了选举”，并负责运行故障转移以使其本地NameNode处于活动状态。故障转移过程类似于上述的手动故障转移：首先，如有必要，将先前的活动节点隔离，然后将本地NameNode转换为活动状态。

部署 Zookeeper 略。

在开始配置自动故障转移之前，应关闭集群。当前无法在集群运行时从手动故障转移设置过渡到自动故障转移设置。



#### 配置自动故障转移

自动故障转移的配置需要在配置中添加两个新参数。在您的hdfs-site.xml文件中，添加：

```xml
 <property>
   <name>dfs.ha.automatic-failover.enabled</name>
   <value>true</value>
 </property>
```

这指定应为自动故障转移设置群集。在您的core-site.xml文件中，添加：

```xml
 <property>
   <name>ha.zookeeper.quorum</name>
   <value>zk1.example.com:2181,zk2.example.com:2181,zk3.example.com:2181</value>
 </property>
```

与本文档前面所述的参数一样，可以通过在配置密钥后缀名称服务ID来基于每个名称服务配置这些设置。例如，在启用了联盟的群集中，可以通过设置dfs.ha.automatic-failover.enabled.my-nameservice-id显式地仅对其中一种名称服务启用自动故障转移。

还可以设置其他几个配置参数来控制自动故障转移的行为。但是，对于大多数安装而言，它们不是必需的。有关详细信息，请参阅特定于配置密钥的文档。



#### 在ZooKeeper中初始化HA状态

添加配置密钥后，下一步是在ZooKeeper中初始化所需的状态。您可以通过从其中一个NameNode主机运行以下命令来做到这一点。

```bash
$ hdfs zkfc -formatZK
```

这将在ZooKeeper中创建一个znode，自动故障转移系统将在其中存储其数据。



#### 使用start-dfs.sh启动集群

由于已在配置中启用了自动故障转移，因此start-dfs.sh脚本现在将在任何运行NameNode的计算机上自动启动ZKFC守护程序。 ZKFC启动时，它们将自动选择一个NameNode激活。



#### 手动启动集群

如果您手动管理群集上的服务，则需要在运行NameNode的每台计算机上手动启动zkfc守护程序。您可以通过运行以下命令启动守护程序：

```bash
$ hdfs --daemon start zkfc
```



#### 安全访问 ZooKeeper

如果运行的是安全群集，则可能需要确保ZooKeeper中存储的信息也受到保护。这可以防止恶意客户端修改ZooKeeper中的元数据或潜在地触发错误的故障转移。 为了保护ZooKeeper中的信息，首先将以下内容添加到您的core-site.xml文件中：

```xml
 <property>
   <name>ha.zookeeper.auth</name>
   <value>@/path/to/zk-auth.txt</value>
 </property>
 <property>
   <name>ha.zookeeper.acl</name>
   <value>@/path/to/zk-acl.txt</value>
 </property>
```

请注意这些值中的“ @”字符-这表明配置不是内联的，而是指向磁盘上的文件。身份验证信息也可以通过CredentialProvider读取（请参阅hadoop-common项目中的CredentialProviderAPI指南）。

第一个配置的文件指定ZooKeeper身份验证列表，格式与ZK CLI使用的格式相同。例如，您可以指定以下内容：

```
digest:hdfs-zkfcs:mypassword
```

其中hdfs-zkfcs是ZooKeeper的唯一用户名，而mypassword是用作密码的一些唯一字符串。

接下来，使用类似以下的命令生成与此身份验证相对应的ZooKeeper ACL：

```bash
$ java -cp $ZK_HOME/lib/*:$ZK_HOME/zookeeper-3.4.2.jar org.apache.zookeeper.server.auth.DigestAuthenticationProvider hdfs-zkfcs:mypassword
output: hdfs-zkfcs:mypassword->hdfs-zkfcs:P/OQvnYyU/nF/mGYvB/xurX8dYs=
```

将“->”字符串后的输出部分复制并粘贴到文件zk-acls.txt中，该文件的前缀为“ digest：”。例如：

```
digest:hdfs-zkfcs:vlUvLnd8MlacsE80rDuu6ONESbM=:rwcda
```

为了使这些ACL生效，您应该按照上述说明重新运行zkfc -formatZK命令。

这样做之后，您可以按照以下步骤从ZK CLI验证ACL：

```
[zk: localhost:2181(CONNECTED) 1] getAcl /hadoop-ha
'digest,'hdfs-zkfcs:vlUvLnd8MlacsE80rDuu6ONESbM=
: cdrwa
```



#### 验证自动故障转移

设置自动故障转移后，应测试其操作。为此，首先找到活动的NameNode。您可以通过访问NameNode Web界面来确定哪个节点处于活动状态–每个节点在页面顶部报告其HA状态。

找到活动的NameNode之后，可能会导致该节点发生故障。例如，您可以使用kill -9 <NN的pid>模拟JVM崩溃。或者，您可以重新启动计算机电源或拔出其网络接口以模拟另一种中断。触发您要测试的中断后，另一个NameNode应在几秒钟内自动变为活动状态。检测故障和触发故障转移所需的时间取决于ha.zookeeper.session-timeout.ms的配置，但默认值为5秒。

如果测试不成功，则可能是配置错误。检查zkfc守护程序以及NameNode守护程序的日志，以便进一步诊断问题。















































