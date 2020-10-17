# ResourceManager 重启

官方文档：https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/ResourceManagerRestart.html

ResourceManager是管理资源并计划在YARN上运行的应用程序的中央机构。因此，它可能是Apache YARN群集中的单点故障。本文档概述了ResourceManager重新启动，此功能可以增强ResourceManager使其在重新启动后仍能正常运行，并且还使ResourceManager的停机时间对最终用户不可见。

ResourceManager有两种重新启动类型：

- **不保留工作的RM重新启动**：此重新启动增强了RM，以将应用程序/尝试状态和其他凭据信息保留在可插拔状态存储中。RM将在重新启动时从状态存储中重新加载此信息，并重新启动以前运行的应用程序。不需要用户重新提交应用程序。
- **保留工作的RM重新启动**：此操作着重于通过结合NodeManagers的容器状态和ApplicationMasters在重新启动时的容器请求来重建RM的运行状态。与不保留工作的RM重新启动的主要区别在于，RM重新启动后，先前运行的应用程序不会被杀死，因此应用程序不会因为RM中断而失去工作。

特征：

- **不保留工作的RM重新启动**

  在不保留工作的RM重新启动时，RM将在客户端提交应用程序时将应用程序元数据（即ApplicationSubmissionContext）保存在可插拔状态存储中，并保存应用程序的最终状态，例如完成状态（失败，中止或终止）。完成），并在应用程序完成时进行诊断。此外，RM还保存凭据（如安全密钥，令牌）以在安全环境中工作。当RM关闭时，只要状态存储中提供了所需的信息（即，应用程序元数据和在安全环境中运行的凭据），则RM重新启动时，便可以从状态存储中获取应用程序元数据并重新提交申请。如果RM在停机之前已经完成（即失败，中止或完成），则RM不会重新提交申请。

  在RM停机期间，NodeManager和客户端将继续轮询RM，直到RM出现为止。当RM启动时，它将通过心跳向正在与之通信的所有NodeManager和ApplicationMaster发送重新同步命令。NM将杀死其所有托管容器并向RM重新注册。这些重新注册的NodeManager与新加入的NM相似。当AM（例如MapReduce AM）接收到重新同步命令时，它们将关闭。RM重新启动并从状态存储加载所有应用程序元数据，凭据并将其填充到内存后，它将为每个尚未完成的应用程序创建一个新尝试（即ApplicationMaster），并照常重新踢该应用程序。如前所述。

- **保留工作的RM重新启动**

  在保留工作的RM重新启动中，RM确保应用程序状态的持久性并在恢复时重新加载该状态，此重新启动主要着重于重建YARN集群的整个运行状态，其中大部分是RM内部中央调度程序的状态。跟踪所有容器的生命周期，应用程序的净空和资源请求，队列的资源使用情况等等。这样，RM无需终止AM并从头开始重新运行应用程序，因为它是在不保留工作的RM重新启动中完成的。应用程序可以简单地与RM重新同步，并从停止的地方恢复。

  RM通过利用从所有NM发送的容器状态来恢复其运行状态。与重新启动的RM重新同步时，NM将不会杀死容器。它继续管理容器，并在重新注册时将容器状态发送给RM。RM通过吸收这些容器的信息来重建容器实例和关联的应用程序的调度状态。同时，由于RM在关闭时可能会丢失未满足的请求，因此AM需要将未完成的资源请求重新发送给RM。使用AMRMClient库与RM通信的应用程序编写者无需担心AM重新发送资源请求到重新同步到RM的部分，因为库本身会自动处理。



## 配置

启用RM重新启动：

```xml
    <property>
      <name>yarn.resourcemanager.recovery.enabled</name>
      <value>true</value>
    </property>
```

配置状态存储以保留RM状态：

```xml
    <property>
      <name>yarn.resourcemanager.store.class</name>
      <value>org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore</value>
    </property>
```

`org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore`基于ZooKeeper的状态存储实现

`org.apache.hadoop.yarn.server.resourcemanager.recovery.FileSystemRMStateStore`基于Hadoop FileSystem的状态存储实现，例如HDFS和本地FS。

 `org.apache.hadoop.yarn.server.resourcemanager.recovery.LeveldbRMStateStore`, 基于LevelDB的状态存储实现。

 `org.apache.hadoop.yarn.server.resourcemanager.recovery.FileSystemRMStateStore`.

- **ZooKeeper based state-store**: 用户可以自由选择任何存储来设置RM重新启动，但必须使用基于ZooKeeper的状态存储来支持RM HA。原因是只有基于ZooKeeper的状态存储才支持隔离机制，以避免出现裂脑情况，即多个RM假定它们处于活动状态并且可以同时编辑状态存储。
- **FileSystem based state-store**: 支持HDFS和基于本地FS的状态存储。不支持隔离机制。
- **LevelDB based state-store**: 与基于HDFS和ZooKeeper的状态存储相比，基于LevelDB的状态存储被认为重量更轻。 LevelDB支持更好的原子操作，每个状态更新的I / O操作更少，文件系统上的文件总数也少得多。不支持隔离机制。



#### 配置基于ZooKeeper的状态存储实现

Hadoop 3.1 与 Hadoop 3.3 的配置有些不一样！！！

```xml
    <property>
      <name>yarn.resourcemanager.zk-address</name>
      <value>ct1.test.bbdops.com:2181,ct2.test.bbdops.com:2181,ct3.test.bbdops.com:2181</value>
    </property>

    <property>
      <name>yarn.resourcemanager.zk-state-store.parent-path</name>
      <value>/rmstore</value>
    </property>
```

配置重试策略状态存储客户端用于连接ZooKeeper服务器。

```xml
    <property>
      <name>yarn.resourcemanager.zk-num-retries</name>
      <value>1000</value>
    </property>

    <property>
      <name>yarn.resourcemanager.zk-retry-interval-ms</name>
      <value>1000</value>
    </property>

    <property>
      <name>yarn.resourcemanager.zk-timeout-ms</name>
      <value>10000</value>
    </property>
```

ACL 配置：

```xml
    <property>
      <name>yarn.resourcemanager.zk-acl</name>
      <value>sasl:rm:rwcda</value>
    </property>
```



>如果RM在启用了保留工作的恢复的情况下重新启动，则更改ContainerId字符串格式。以前是这样的格式：Container_ {clusterTimestamp} _ {appId} _ {attemptId} _ {containerId}，例如容器_1410901177871_0001_01_000005。
>
>现在已更改为：Container_e {epoch} _ {clusterTimestamp} _ {appId} _ {attemptId} _ {containerId}，例如Container_e17_1410901177871_0001_01_000005。
>
>在此，附加时期号是单调递增的整数，其从0开始，并且每次RM重新启动时都增加1。如果时期号为0，则将其省略，并且containerId字符串格式与以前相同。





















