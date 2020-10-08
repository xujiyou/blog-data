# HDFS 架构

官方文档：https://hadoop.apache.org/docs/r3.3.0/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html

Hadoop 分布式文件系统（HDFS）是一种在商业硬件上运行的分布式文件系统，它与现代的分布式文件系统有很多相似之处。但是，与其他文件系统的区别也很明显，HDFS 具有高度的容错能力，旨在部署在低成本的硬件上。HDFS 提供了对数据的高吞吐量访问，适用于具有大数据级别的程序。HDFS 放宽了 POSIX的要求，以实现对数据的流式访问。

HDFS 最初是 Apache Nutch Web 搜索引擎项目的基础结构而构建的，现在 HDFS 是  Apache Hadoop Core 项目的一部分。



## 设计目标

#### 应对硬件故障

硬件故障是正常现象，而非例外。HDFS 实例可能包含数百上千个服务器，每个服务器都储存一部分数据，存在大量组件，并且每个组件都有可能存在故障，因此，检测故障并快速，自动地从故障中恢复是 HDFS 的核心目标。

#### 流式数据访问

依赖与 HDFS 的应用程序需要对数据进行流式访问。HDFS 被设计用于批处理，而不是用户交互使用，重点在于数据访问的高吞吐量，而不是数据访问的低延迟，POSIX 提出的很多操作对 HDFS 来说不是硬性要求，HDFS 牺牲了一些 POSIX 语意来提高数据吞吐量。

#### 大数据集

HDFS 中储存了大量数据，HDFS 中典型的文件大小为 GB 到 TB，因此，HDFS 被做成适合大文件，应该提供较大的网络带宽。

#### 简单的一致性模型

基于 HDFS 的应用程序需要文件一次写入多次读取，一旦创建、写入和关闭文件，除了追加和截断外，无需更改，支持将内容追加到文件末尾 ，而不能在任意点更新，改假设简化了数据一致化问题并实现了高吞吐量，MapReduce应用程序或Web爬网程序应用程序非常适合此模型。

#### 移动计算而不是移动数据

如果应用程序在具有数据的服务器上执行，效率会更高，当数据量特别大时更是如此，这样可以更大程度的减少网络拥塞，以提高系统的吞吐量。HDFS 为应用程序提供了接口，使他们自己在更靠近数据。

####  异构硬件和软件平台的可移植性

HDFS被设计为可从一个平台轻松移植到另一个平台。这有助于将HDFS广泛用作大量应用程序的首选平台。



## NameNode 与 DataNode

HDFS具有主/从体系结构。 HDFS群集由单个NameNode和管理文件系统名称空间并控制客户端对文件的访问的主服务器组成。此外，还有许多数据节点，通常是集群中每个节点一个，用于管理与它们所运行的节点相连的存储。 HDFS公开了文件系统名称空间，并允许用户数据存储在文件中。在内部，文件被分成一个或多个块，这些块存储在一组DataNode中。 NameNode执行文件系统名称空间操作，例如打开，关闭和重命名文件和目录。它还确定块到DataNode的映射。数据节点负责处理来自文件系统客户端的读写请求。 DataNode还根据NameNode的指令执行块创建，删除和复制。

HDFS 架构图

![HDFS Architecture](../../resource/hdfsarchitecture.png)

群集中单个NameNode的存在极大地简化了系统的体系结构。 NameNode是所有HDFS元数据的仲裁器和存储库。该系统的设计方式是，用户数据永远不会流过NameNode。



## 文件系统命名空间

HDFS支持传统的分层文件组织。用户或应用程序可以创建目录并将文件存储在这些目录中。文件系统名称空间层次结构与大多数其他现有文件系统类似；可以创建和删除文件，将文件从一个目录移动到另一个目录或重命名文件。 HDFS支持用户配额和访问权限。 HDFS不支持硬链接或软链接。但是，HDFS体系结构并不排除实现这些功能。

尽管HDFS遵循文件系统的命名约定，但保留了某些路径和名称（例如/.reserved和.snapshot）。透明加密和快照等功能使用保留的路径。 NameNode维护文件系统名称空间。对文件系统名称空间或其属性的任何更改均由NameNode记录。

应用程序可以指定应由HDFS维护的文件副本的数量。文件的副本数称为该文件的复制因子。此信息由NameNode存储。



## 数据副本

HDFS旨在可靠地在大型群集中的计算机之间存储非常大的文件。它将每个文件存储为一系列块。复制文件的块是为了容错。块大小和复制因子是每个文件可配置的。

文件中除最后一个块外的所有块都具有相同的大小，而在添加了对可变长度块的支持后，用户可以在不填充最后一个块的情况下启动新块，而不用配置的块大小填充最后一个块。

应用程序可以指定文件的副本数。复制因子可以在文件创建时指定，以后可以更改。 HDFS中的文件只能写入一次（追加和截断除外），并且在任何时候都只能具有一个写入器。

NameNode做出有关块复制的所有决定。它定期从群集中的每个DataNode接收心跳信号和Blockreport。收到心跳信号表示DataNode正常运行。 Blockreport包含DataNode上所有块的列表。

![HDFS DataNodes](../../resource/hdfsdatanodes.png)

#### 副本位置

复制副本的位置对于HDFS的可靠性和性能至关重要。优化副本放置将HDFS与大多数其他分布式文件系统区分开来。此功能需要大量调整和经验。机架感知的副本放置策略的目的是提高数据可靠性，可用性和网络带宽利用率。副本放置策略的当前实现是朝这个方向的第一步。实施此策略的短期目标是在生产系统上对其进行验证，进一步了解其行为，并为测试和研究更复杂的策略奠定基础。

大型HDFS实例在通常分布在许多机架上的计算机群集上运行。不同机架中的两个节点之间的通信必须通过交换机进行。在大多数情况下，同一机架中的计算机之间的网络带宽大于不同机架中的计算机之间的网络带宽。

NameNode通过Hadoop机架感知中概述的过程确定每个DataNode所属的机架ID。一个简单但非最佳的策略是将副本放置在唯一的机架上。这样可以防止在整个机架发生故障时丢失数据，并允许在读取数据时使用多个机架的带宽。此策略在群集中均匀分布副本，这使得在组件故障时轻松平衡负载成为可能。但是，此策略增加了写入成本，因为写入需要将块传输到多个机架。

在常见情况下，当复制因子为3时，HDFS的放置策略是：如果写入器位于数据节点上，则将一个副本放置在本地计算机上；否则，将与写入器位于同一机架的随机数据节点放置在本地计算机上，将另一个副本放置在本地计算机上。一个节点（位于不同（远程）机架中），最后一个节点位于同一远程机架中的另一个节点上。此策略减少了机架间的写流量，通常可以提高写性能。机架故障的机会远少于节点故障的机会。此策略不会影响数据的可靠性和可用性保证。但是，它不会减少读取数据时使用的总网络带宽，因为一个块仅放置在两个唯一的机架中，而不是三个。使用此策略，块的副本不会在机架上均匀分布。两个副本位于一个机架的不同节点上，其余副本位于另一个机架之一的节点上。此策略可提高写入性能，而不会影响数据可靠性或读取性能。

如果复制因子大于3，则随机确定第4个及以下副本的位置，同时将每个机架的副本数量保持在上限以下 `(replicas - 1) / racks + 2`。

因为NameNode不允许DataNode具有同一块的多个副本，所以创建的副本的最大数量是当时DataNode的总数。

将对存储类型和存储策略的支持添加到HDFS后，除了上述机架识别之外，NameNode还考虑了该策略的副本放置位置。首先，NameNode基于机架识别来选择节点，然后检查候选节点是否具有与文件关联的策略所需的存储。如果候选节点不具有存储类型，则NameNode将寻找另一个节点。如果在第一条路径中找不到足够的节点来放置副本，则NameNode将在第二条路径中查找具有备用存储类型的节点。



#### 副本选择

为了最大程度地减少全局带宽消耗和读取延迟，HDFS尝试满足最接近读取器的副本的读取请求。如果在与读取器节点相同的机架上存在一个副本，则该副本应优先满足读取请求。如果HDFS群集跨越多个数据中心，则驻留在本地数据中心中的副本比任何远程副本都优先。



#### 安全模式

安全模式启动时，NameNode进入一个特殊的状态，称为安全模式。当NameNode处于安全模式状态时，不会发生数据块的复制。 NameNode从数据节点接收心跳和Blockreport消息。 Blockreport包含DataNode托管的数据块列表。每个块都有指定的最小副本数。当已使用NameNode检入该数据块的最小副本数时，该块被视为已安全复制。在可配置百分比的安全复制数据块中通过NameNode签入（再加上30秒）后，NameNode退出安全模式状态。然后，它确定仍少于指定数量的副本的数据块列表（如果有）。然后，NameNode将这些块复制到其他DataNode。



## 文件系统元数据的持久化

HDFS命名空间由NameNode存储。 NameNode使用称为EditLog的事务日志来永久记录文件系统元数据发生的所有更改。例如，在HDFS中创建一个新文件将导致NameNode将一条记录插入到EditLog中，以表明这一点。同样，更改文件的复制因子会导致将新记录插入到EditLog中。 NameNode使用其本地主机OS文件系统中的文件来存储EditLog。整个文件系统名称空间（包括块到文件的映射和文件系统属性）存储在名为FsImage的文件中。 FsImage也作为文件存储在NameNode的本地文件系统中。

NameNode在内存中保留整个文件系统名称空间和文件Blockmap的图像。当NameNode启动时，或者由可配置的阈值触发检查点时，它将从磁盘读取FsImage和EditLog，将EditLog中的所有事务应用于FsImage的内存中表示形式，并将此新版本刷新为磁盘上的新FsImage。然后，它可以截断旧的EditLog，因为其事务已应用于持久性FsImage。此过程称为检查点。检查点的目的是通过对文件系统元数据进行快照并将其保存到FsImage来确保HDFS对文件系统元数据具有一致的视图。即使读取FsImage效率很高，但直接对FsImage进行增量编辑效率也不高。我们无需为每个编辑修改FsImage，而是将编辑保留在Editlog中。在检查点期间，来自Editlog的更改将应用于FsImage。可以在以秒为单位的给定时间间隔（dfs.namenode.checkpoint.period）或累积一定数量的文件系统事务之后（dfs.namenode.checkpoint.txns）触发检查点。如果同时设置了这两个属性，则要达到的第一个阈值将触发检查点。

DataNode将HDFS数据存储在其本地文件系统中的文件中。 DataNode不了解HDFS文件。它将每个HDFS数据块存储在其本地文件系统中的单独文件中。 DataNode不会在同一目录中创建所有文件。而是使用启发式方法确定每个目录的最佳文件数，并适当创建子目录。在同一目录中创建所有本地文件不是最佳选择，因为本地文件系统可能无法有效地支持单个目录中的大量文件。当DataNode启动时，它将扫描其本地文件系统，生成与每个本地文件相对应的所有HDFS数据块的列表，并将此报告发送到NameNode。该报告称为Blockreport。



## 通讯协议

所有HDFS通信协议都位于TCP / IP协议之上。客户端建立与NameNode计算机上可配置TCP端口的连接。它将ClientProtocol与NameNode进行通信。 DataNode使用DataNode协议与NameNode对话。远程过程调用（RPC）抽象包装了客户端协议和DataNode协议。按照设计，NameNode永远不会启动任何RPC。相反，它仅响应由DataNode或客户端发出的RPC请求。



## 健壮性

HDFS的主要目标是即使在出现故障的情况下也能可靠地存储数据。三种常见的故障类型是NameNode故障，DataNode故障和网络分区。



#### 数据磁盘故障，心跳和复制

每个DataNode定期向NameNode发送心跳消息。网络分区可能导致一部分DataNode失去与NameNode的连接。 NameNode通过缺少心跳消息来检测到这种情况。 NameNode将没有最近心跳的DataNode标记为无效，并且不向其转发任何新的IO请求。已注册到失效DataNode的任何数据不再可用于HDFS。 DataNode死亡可能导致某些块的复制因子降至其指定值以下。 NameNode不断跟踪需要复制的块，并在必要时启动复制。由于许多原因，可能需要进行重新复制：DataNode可能不可用，副本可能损坏，DataNode上的硬盘可能发生故障或文件的复制因子可能增加。

标记DataNode失效的超时时间保守地长（默认情况下超过10分钟），以避免由DataNode的状态震荡引起的复制风暴。用户可以设置较短的时间间隔以将DataNode标记为过时，并通过配置来避免对性能敏感的工作负载进行读和/或写时出现过时的节点。



#### 集群再平衡

HDFS体系结构与数据重新平衡方案兼容。如果DataNode的可用空间低于某个阈值，则方案可能会自动将数据从一个DataNode移到另一个DataNode。如果对特定文件的需求突然增加，则方案可能会动态创建其他副本并重新平衡群集中的其他数据。这些类型的数据重新平衡方案尚未实现。



#### 数据的完整性

从DataNode提取的数据块可能会损坏。由于存储设备故障，网络故障或软件故障，可能会导致这种损坏。 HDFS客户端软件对HDFS文件的内容执行校验和检查。客户端创建HDFS文件时，它将计算文件每个块的校验和，并将这些校验和存储在同一HDFS名称空间中的单独的隐藏文件中。客户端检索文件内容时，它将验证从每个DataNode接收到的数据是否与关联的校验和文件中存储的校验和匹配。如果不是，则客户端可以选择从另一个具有该块副本的DataNode中检索该块。



#### 元数据磁盘故障

FsImage和EditLog是HDFS的中央数据结构。这些文件损坏可能导致HDFS实例无法正常运行。因此，可以将NameNode配置为支持维护FsImage和EditLog的多个副本。 FsImage或EditLog的任何更新都会导致每个FsImages和EditLogs同步更新。 FsImage和EditLog的多个副本的这种同步更新可能会降低NameNode可以支持的每秒名称空间事务处理的速度。但是，这种降级是可以接受的，因为即使HDFS应用程序本质上是数据密集型的，但它们也不是元数据密集型的。当NameNode重新启动时，它将选择要使用的最新一致的FsImage和EditLog。

增强抗故障能力的另一种方法是使用多个NameNode来启用高可用性，这些NameNode可以在NFS上使用共享存储，也可以使用分布式编辑日志（称为Journal）。推荐使用后者。



#### 快照

快照支持在特定时间存储数据副本。快照功能的一种用法可能是将损坏的HDFS实例回滚到以前已知的良好时间点。



## 数据组织

#### 数据块

HDFS旨在支持非常大的文件。与HDFS兼容的应用程序是处理大型数据集的应用程序。这些应用程序仅写入一次数据，但读取一次或多次，并要求以流速度满足这些读取要求。 HDFS支持文件上一次写入多次读取的语义。 HDFS使用的典型块大小为128 MB。因此，HDFS文件被切成128 MB的块，并且如果可能的话，每个块将驻留在不同的DataNode上。

#### 复制流

当客户端将数据写入复制因子为3的HDFS文件时，NameNode使用复制目标选择算法检索DataNode列表。该列表包含将托管该块副本的DataNode。然后，客户端写入第一个DataNode。第一个DataNode开始分批接收数据，将每个部分写入其本地存储库，然后将该部分传输到列表中的第二个DataNode。第二个DataNode依次开始接收数据块的每个部分，将该部分写入其存储库，然后将该部分刷新到第三个DataNode。最后，第三个DataNode将数据写入其本地存储库。因此，DataNode可以从流水线中的前一个接收数据，同时将数据转发到流水线中的下一个。因此，数据从一个DataNode流水到下一个。



## 集群管理

可以通过许多不同的方式从应用程序访问HDFS。 HDFS本身就为应用程序提供了FileSystem Java API。也提供此Java API和REST API的C语言包装器。此外，HTTP浏览器还可以用于浏览HDFS实例的文件。通过使用NFS网关，HDFS可以作为客户端本地文件系统的一部分安装。

#### FS Shell

HDFS允许以文件和目录的形式组织用户数据。它提供了一个称为FS shell的命令行界面，该界面可让用户与HDFS中的数据进行交互。该命令集的语法类似于用户已经熟悉的其他shell（例如bash，csh）。以下是一些示例操作/命令对：

| Action                                                 | Command                                  |
| :----------------------------------------------------- | :--------------------------------------- |
| Create a directory named `/foodir`                     | `bin/hadoop dfs -mkdir /foodir`          |
| Remove a directory named `/foodir`                     | `bin/hadoop fs -rm -R /foodir`           |
| View the contents of a file named `/foodir/myfile.txt` | `bin/hadoop dfs -cat /foodir/myfile.txt` |

FS Shell针对需要脚本语言与存储的数据进行交互的应用程序。



#### DFSAdmin

DFSAdmin命令集用于管理HDFS群集。这些是仅由HDFS管理员使用的命令。以下是一些示例操作/命令对：

| Action                                   | Command                             |
| :--------------------------------------- | :---------------------------------- |
| Put the cluster in Safemode              | `bin/hdfs dfsadmin -safemode enter` |
| Generate a list of DataNodes             | `bin/hdfs dfsadmin -report`         |
| Recommission or decommission DataNode(s) | `bin/hdfs dfsadmin -refreshNodes`   |



#### 浏览器接口

典型的HDFS安装会将Web服务器配置为通过可配置的TCP端口公开HDFS命名空间。这允许用户使用Web浏览器浏览HDFS命名空间并查看其文件的内容。



## 节约空间

#### 文件删除与恢复

如果启用垃圾桶配置，则不会立即从HDFS中删除由FS Shell删除的文件。而是，HDFS将其移动到回收站目录（每个用户在/ user / <用户名> /。Trash下都有自己的回收站目录）。只要文件保留在垃圾桶中，就可以快速恢复。

最新删除的文件被移动到当前的废纸directory目录（/ user / <用户名> /。Trash / Current），并且在可配置的间隔内，HDFS创建检查点（在/ user / <用户名> /。Trash / <date>下）用于当前废纸directory目录中的文件，并在过期时删除旧的检查点。有关垃圾的检查点，请参见FS shell的expunge命令。

在垃圾桶中到期后，NameNode将从HDFS命名空间中删除该文件。文件的删除导致与文件关联的块被释放。请注意，在用户删除文件的时间与HDFS中相应的可用空间增加的时间之间可能会有明显的时间延迟。

下面是一个示例，它将显示FS Shell如何从HDFS删除文件。我们在目录delete下创建了2个文件（test1和test2）

```bash
$ hadoop fs -mkdir -p delete/test1
$ hadoop fs -mkdir -p delete/test2
$ hadoop fs -ls delete/
Found 2 items
drwxr-xr-x   - hadoop hadoop          0 2015-05-08 12:39 delete/test1
drwxr-xr-x   - hadoop hadoop          0 2015-05-08 12:40 delete/test2
```

我们将删除文件test1。以下评论显示该文件已移至“废纸directory”目录。

```bash
$ hadoop fs -rm -r delete/test1
Moved: hdfs://localhost:8020/user/hadoop/delete/test1 to trash at: hdfs://localhost:8020/user/hadoop/.Trash/Current
```

现在我们要使用skipTrash选项删除文件，该选项不会将文件发送到Trash。它将从HDFS中完全删除。

```bash
$ hadoop fs -rm -r -skipTrash delete/test2
Deleted delete/test2
```

现在我们可以看到垃圾箱目录仅包含文件test1。

```bash
$ hadoop fs -ls .Trash/Current/user/hadoop/delete/
Found 1 items\
drwxr-xr-x   - hadoop hadoop          0 2015-05-08 12:39 .Trash/Current/user/hadoop/delete/test1
```

#### 减少复制因子

当减少文件的复制因子时，NameNode选择可以删除的多余副本。下一个心跳将此信息传输到DataNode。然后，DataNode删除相应的块，并且相应的可用空间出现在群集中。同样，在setReplication API调用完成与群集中的可用空间出现之间可能会有时间延迟。











