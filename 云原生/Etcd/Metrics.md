# Metrics

官方文档：https://etcd.io/docs/v3.4.0/metrics/

etcd 使用 [Prometheus](https://prometheus.io/) 来收集 Metrics，Metrics 可以用来实时监控和 debug。etcd不会保留其指标；如果成员重新启动，则指标将被重置。

/metrics 这个地址可以查看 etcd 的所有 Metrics：

```bash
$ curl --cacert /etc/etcd/cert/etcd/ca.pem https://fueltank-1:2379/metrics
```



## 服务端指标

这些指标都带有前缀 `etcd_server_`

| Name                      | 描述                             | 类型    |
| :------------------------ | :------------------------------- | :------ |
| has_leader                | Leader是否存在。1存在，0不存在。 | Gauge   |
| Leader_changes_seen_total | leader的变更次数。               | Counter |
| proposal_committed_total  | 提交的总数。                     | Gauge   |
| proposal_applied_total    | 已应用提交总数。                 | Gauge   |
| proposal_pending          | 当前待处理的数量。               | Gauge   |
| proposal_failed_total     | 失败提交总数。                   | Counter |



## 硬盘指标

这些指标都以开头`etcd_disk_`。

| Name                            | 描述                       | 类型      |
| :------------------------------ | :------------------------- | :-------- |
| wal_fsync_duration_seconds      | Wal调用的fsync的延迟分布   | Histogram |
| backend_commit_duration_seconds | 后端调用的提交的延迟分布。 | Histogram |



## 网络指标

这些指标都带有前缀 `etcd_network_`

| Name                             | 描述                                   | 类型          |
| :------------------------------- | :------------------------------------- | :------------ |
| peer_sent_bytes_total            | 发送给ID为的对等方的字节总数`To`。     | Counter(To)   |
| peer_received_bytes_total        | 从对等方收到的ID为的字节总数`From`。   | Counter(From) |
| peer_sent_failures_total         | 来自ID为的对等方的发送失败总数`To`。   | Counter(To)   |
| peer_received_failures_total     | 来自ID为的对等方的接收失败总数`From`。 | Counter(From) |
| peer_round_trip_time_seconds     | 同伴之间的往返时间直方图。             | Histogram     |
| client_grpc_sent_bytes_total     | 发送给grpc客户端的字节总数。           | Counter       |
| client_grpc_received_bytes_total | 接收到grpc客户端的字节总数。           | Counter       |



## gRPC 请求

查看：https://github.com/grpc-ecosystem/go-grpc-prometheus



## etcd_debugging namespace metrics

`etcd_debugging`前缀下的指标用于调试。它们非常依赖于实现且易变。在新的etcd发行版中，可能会更改或删除它们而没有任何警告。当某些指标`etcd`变得更稳定时，它们可能会移到前缀。



## 快照指标

| Name                                 | 描述                       | 类型      |
| :----------------------------------- | :------------------------- | :-------- |
| snapshot_save_total_duration_seconds | 快照调用的保存的总延迟分布 | Histogram |



## Prometheus 提供的指标

| Name             | 描述                       | 类型  |
| :--------------- | :------------------------- | :---- |
| process_open_fds | 打开文件描述符的数量。     | Gauge |
| process_max_fds  | 打开文件描述符的最大数量。 | Gauge |