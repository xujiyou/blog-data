# Raft 分布式一致性协议原理

Raft 是一个相对比较简单易于理解的分布式一致性协议，现在工业界使用的也比较多，包括 Etcd、TiDB 等都在使用。



## Raft 算法中的节点角色

**1. Leader（领导者）**：处理所有客户端交互，日志复制等，一般一次只有一个Leader；

**2. Follower（追随者）**：响应 Leader 的日志同步请求，响应 Candidate 的邀票请求，以及把客户端请求到 Follower 的事务转发（重定向）给 Leader；

**3. Candidate（候选者）**：负责选举投票，集群刚启动或者 Leader 宕机时，角色为 Follower 的节点将转为 Candidate 并发起选举，选举胜出（获得超过半数节点的投票）后，从 Candidate 转为 Leader 角色；



## Raft 算法分解

Raft 算法总共分为三部分：

**1. Leader election** 当leader宕机或者集群创建时，需要选举一个新的Leader

**2. Log replication：**Leader接收来自客户端的请求并将其以日志的形式复制到集群中的其它节点，并且强制要求其它节点的日志和自己保持一致

**3. Safety：**如果有任何节点已经应用了一个确定的日志条目到它的状态机中，那么其它服务节点不能在同一个日志索引位置应用一个不用的指令



## Raft 选举步骤

根据 Raft 协议，一个应用 Raft 协议的集群在刚启动时，所有节点状态都是Follower态，由于没有Leader，Follower 无法与 Leader 保持心跳（heart beat），Follower等待心跳超时（**每个Follower的心跳超时时间不一样**），Followers 会认为 Leader 已经 down 掉。最先超时的Follower进而转为 Candidate 状态，然后，Candidate 将向集群中的其它节点请求投票，同意自己升级为 Leader，如果 Candidate 收到超过半数节点的投票（N/2+1），它将获胜成为 Leader。

角色选举详细流程如下：

**第一阶段：都是 Follower 状态**

根据 Raft 协议，一个应用 Raft 协议的集群在刚启动时，所有节点状态都是Follower态，由于没有Leader，Follower 无法与 Leader 保持心跳（heart beat），Follower等待心跳超时（**每个Follower的心跳超时时间不一样**），Followers 会认为 Leader 已经 down 掉。最先超时的Follower进而转为 Candidate 状态，然后，Candidate 将向集群中的其它节点请求投票，同意自己升级为 Leader，如果 Candidate 收到超过半数节点的投票（N/2+1），它将获胜成为 Leader。

**第二阶段：从 Follower 状态转换为Candidate，并发起投票**

由于没有 Leader，Followers 无法与 Leader 保持心跳（heart beat），节点启动后在一个选举定时器周期内未收到心跳和投票请求，则状态转变为 Candidate 状态、Term 自增，并向集群中所有节点发送投票请求并且重置选举定时器。

注意：每个节点选举定时器超时时间都在 100 ～ 500 ms之内，且不一致。因此，可以避免所有的Follower同时转化为 Candidate状态，换言之，最先转为 Candidate 并发起投票请求的节点将具有成为 Leader 的先发优势。

**第三阶段：投票策略**

Follower 节点收到投票请求后会根据以下情况决定是否接受投票请求：

- 请求节点的 Term 大于自己的 Term，且自己尚未投票给其它节点，则接受请求，把票投给 Candidate 节点；
- 请求节点的 Term 小于自己的 Term，且自己尚未投票，则拒绝请求，将票投给自己。

**第四阶段：Candidate 转换为 Leader**

经过一轮选举后，正常情况，会有一个 Candidate 节点收到超过半数（N/2+1）其它节点的投票，那么它将胜出并升级为 Leader 节点，然后定时发送心跳给其它节点，其它节点会转化为 Follower 节点并与 Leader 保持同步，如此，本轮选举结束。如果一轮选举中，Candidate 节点收到的投票没有超过半数，那么将进行下一轮选举。



## Raft 日志同步

一个 Raft 集群中只有 Leader 节点能够处理客户端的请求，如果客户端的请求发到了 Follower 节点，Follower 将会把请求重定向到 Leader。

客户端的每一个请求都包含一条被复制到状态机执行的指令。Leader 把这条指令作为一条新的日志条目（Entry）附加到日志中去，然后并行的将附加条目发送给 Followers，让它们复制这条日志条目。

当这条日志条目被 Followers 安全的复制，Leader 会应用这条日志条目到它的状态机中，然后把执行的结果返回给客户端。

如果 Follower 崩溃或者运行缓慢，再或者是网络丢包，Leader 会不断的重复尝试附加日志条目（尽管已经回复了客户端）直到所有的 Follower 最终都存储了所有的日志条目，**确保强一致性**。

**第一阶段：客户端请求提交到 Leader**

Leader 收到客户端请求：如存储一个数据：5；Leader 收到请求后，会将它作为日志条目（Entry）写入本地日志中。此时该 Entry 是未提交状态（uncommitted），Leader并不会更新本地数据，因此它是不可读的。

**第二阶段：Leader 将 Entry 发送到其它Follower**

Leader 与 Followers 之间保持心跳联系，跟心跳 Leader 将追加的 Entry（AppendEntries） 并行的发送到其它 Follower 节点，并让它们复制这条日志条目，这一过程我们称为：复制（Replication）。

1. 为什么 Leader 向 Follower 发送的 Entry 是 AppendEntries 呢？

因为 Leader 与 Follower 的心跳是周期性的，而一个周期 Leader 可能接收到客户端的多个请求，因此，随 心跳向 Followers 发送的大概率是多个 Entry，即 AppendEntries。在本例中为了简单，只有一条请求，自然 只有一个 Enrety。

2. Leader 向 Followers 发送的不仅仅是追加的 Entry （AppendEntries）

在发送追加日志条目的时候，Leader 会把新日志条目之前的条目索引（前一个日志条目）位置（prevLogIndex）和Leader任期号（term）包含在里边。如果 Follower 在它的日志中找不到包含相同索引位置和任期号的条目，那么它会拒接这个新的日志条目。因为出现这种情况说明 Follower 和 Leader 是不一致的。

3. 如何解决 Leader 和 Follower 不一致的问题？

在正常情况下，Leader 和 Follower 的日志保持一致，所以追加日志的一致性从来不会失败。然后，Leader 和 Follower 的一系列崩溃情况下会使它们的日志处于不一致的状态。Follower 可能会丢失一些在新的 Leader 中有的日志条目，它也可能拥有一些 Leader 没有的日志条目，或者两者都有发生。丢失或者多出的日志条目可能会持续多个任期。

要使 Follower 的日志与 Leader 恢复一致，Leader 必须找到最后两者达成一致的地方，然后删除从那个节点之后的所有日志，发送自己的日志给 Follower。所有的这些操作都在进行附加日志一致性检查时完成。

Leader 节点针对每个 Follower 节点维护了一个 nextIndex，这表示下一个需要发送给 Follower 的日志条目的索引地址。当一个 Leader 刚获得权力的时候，它初始化所有的 nextIndex 值为自己的最后一条日志的 index + 1。如果一个 Follower 日志和 Leader 不一致，那么在下一次附加日志的时候就会检查失败。在被 Follower 拒绝之后，Leader 就会减小该 Follower 对应的 nextIndex 值并进行重试（即回溯）。

最终 nextIndex 会在某个位置使得 Leader 和 Follower 的日志达成一致。当这种情况发生，附加日志就会成功，这时就会把 Follower 冲突的日志条目全部删除并且附加上 Leader 的日志。一旦附加成功，那么 Follower 的日志就会和 Leader 保持一致，并且在接下来的任期里一致继续保持。

**第三阶段：Leader 等待 Followers 回应**

Followers 接收到 Leader 发来的复制请求后，有两种可能的回应：

- 写入本地日志，返回 Success
- 一致性检查失败，拒绝写入，返回 false。原因和解决办法上面已经详细说明。

注：此时该 Entry 的状态也是未提交（uncommitted）。完成上述步骤后，Followers 会向 Leader 发出回应 - success，当 Leader 收到大多数 Followers 的回应后，会将第一阶段写入的 Entry 标记为提交状态（committed），并把这条日志条目应用到它的状态机中。

**第四阶段：Leader回应客户端**

完成前三个阶段后，Leader 会回应客户端 - OK，写操作成功。

**第五阶段：Leader 通知 Followers Entry 已提交**

Leader 回应客户端后，将随着下一个心跳通知 Followers，Followers 收到通知后也会将 Entry 标记为提交状态。至此，Raft 集群超过半数节点已经达到一致状态，可以确保强一致性。需要注意的是，由于网络、性能、故障等各种原因导致的“反应慢”、“不一致”等问题的节点，也会最终与 Leader 达成一致。



## Raft 安全性保证

前面的章节里描述了 Raft 算法是如何选举 Leader 和 日志复制。然而，到目前为止描述的机制并不能充分保证每一个状态机会按照相同的顺序执行相同的指令。例如：一个 Follower 可能处于不可用的状态，同时 Leader 已经提交了若干的日志条目；然后这个 Follower 恢复（尚未与 Leader 达成一致）而 Leader 故障，如果该 Follower 被选举为 Leader 并且覆盖这些日志条目，就会出现问题：不同的状态机执行不同的指令序列。

鉴于此，在 Leader 选举的时候需要增加一些限制来完善 Raft 算法。这些限制可保证任何的 Leader 对于给定的任期号（Term），都拥有之前任期的所有被提交的日志条目（所谓 Leader 的完整特性）。



## 总结

TiDB， etcd 等都在使用 Raft 协议。只要你的服务有一致性高可用需求，都可以考虑使用 Raft 协议

Raft算法具备强一致、高可靠、高可用等优点，具体体现在：

**强一致性**：虽然所有节点的数据并非实时一致，但 Raft 算法保证 Leader 节点的数据最全，同时所有请求都由Leader 处理，所以在客户端角度看是强一致性的。

**高可靠性**：Raft算法保证了Committed的日志不会被修改，状态机只应用 Committed 的日志，所以当客户端收到请求成功即代表数据不再改变。Committed 日志在大多数节点上冗余存储，少于一半的磁盘故障数据不会丢失。

**高可用性**：从Raft算法原理可以看出，选举和日志同步都只需要大多数的节点正常互联即可，所以少量节点故障或网络异常不会影响系统的可用性。即使 Leader 故障，在选举超时到期后，集群自发选举新 Leader，无需人工干预，不可用时间极小。但 Leader 故障时存在重复数据问题，需要业务去重或幂等性保证。

**高性能**：与必须将数据写到所有节点才能返回客户端成功的算法相比，Raft 算法只需要大多数节点成功即可，少量节点处理缓慢不会延缓整体系统运行。









