# Observer NameNode

官方文档：https://hadoop.apache.org/docs/r3.3.0/hadoop-project-dist/hadoop-hdfs/ObserverNameNode.html

本指南概述了HDFS Observer NameNode功能以及如何在典型的启用HA的群集中进行配置/安装。有关详细的技术设计概述，请查看HDFS-12943随附的文档。

在启用HA的HDFS群集中，有一个活动的NameNode和一个或多个Standby NameNode。 Active NameNode负责服务所有客户端请求，而Standby NameNode则通过保留JournalNodes的编辑日志以及来自所有DataNode的块报告来保留有关命名空间的最新信息以及块位置信息。这种体系结构的一个缺点是，活动NameNode可能是一个瓶颈，并且可能因客户端请求而过载，尤其是在繁忙的群集中。

HDFS Observer NameNode的一致性读取功能通过引入一种称为Observer NameNode的新型NameNode来解决上述问题。与备用NameNode相似，Observer NameNode保持有关名称空间和块位置信息的最新信息。此外，它还可以提供一致的读取服务，例如Active NameNode。由于读请求在典型环境中占多数，因此可以帮助平衡NameNode流量并提高整体吞吐量。



## 架构

在新架构中，HA集群可以由处于3种不同状态的名称节点组成：活动，备用和观察者。状态转换可以在活动和备用，备用和观察者之间发生，但不能直接在活动和观察者之间发生。

为了确保单个客户端内的写入后读取一致性，在RPC标头中引入了一个状态ID，该状态ID是使用NameNode中的事务ID实现的。客户端通过活动NameNode执行写入时，它将使用NameNode中的最新事务ID更新其状态ID。在执行后续读取时，客户端将此状态ID传递给Observer NameNode，后者将检查其自身的事务ID，并确保在为读取请求提供服务之前，其自身的事务ID已追上请求的状态ID。这样可以确保从单个客户端“读取自己的写入”语义。在下面的“维护客户端一致性”部分中讨论了面对带外通信时维护多个客户端之间的一致性。

编辑日志尾部对于Observer NameNode至关重要，因为它直接影响在Active NameNode中应用事务与在Observer NameNode中应用事务之间的等待时间。引入了一种新的编辑日志尾部机制，称为“编辑尾迹快速路径”，以显着减少此延迟。它建立在现有的正在进行的编辑日志尾部功能的基础上，并进行了进一步的改进，例如基于RPC的尾部（而不是HTTP），JournalNode上的内存中缓存等。有关更多详细信息，请参见随附的设计文档。到HDFS-13150。

还引入了新的客户端代理提供程序。继承现有的ConfiguredFailoverProxyProvider的ObserverReadProxyProvider应该用于替换后者，以启用对Observer NameNode的读取。提交客户端读取请求时，代理提供程序将首先尝试集群中可用的每个Observer NameNode，只有在所有前者都失败的情况下，才回退到Active NameNode。同样，引入了ObserverReadProxyProviderWithIPFailover来替换IP故障转移设置中的IPFailoverProxyProvider。





















