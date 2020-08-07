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



















