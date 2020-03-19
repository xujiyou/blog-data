# Etcd 中的各种版本

在 etcdctl 的返回中，我们可以看到各种版本信息：

```bash
$ etcdctl put foo bar
$ etcdctl get foo -w json | json_pp
{
   "count" : 1,
   "header" : {
      "raft_term" : 2,
      "member_id" : 10276657743932975437,
      "cluster_id" : 14841639068965178418,
      "revision" : 2
   },
   "kvs" : [
      {
         "value" : "YmFy",
         "key" : "Zm9v",
         "create_revision" : 2,
         "mod_revision" : 2,
         "version" : 1
      }
   ]
}
```

这个响应的类型为 `RangeResponse` ，源码在 `etcdserver/etcdserverpb/rpc.proto`：

```protobuf
message RangeResponse {
  ResponseHeader header = 1;
  // kvs is the list of key-value pairs matched by the range request.
  // kvs is empty when count is requested.
  repeated mvccpb.KeyValue kvs = 2;
  // more indicates if there are more keys to return in the requested range.
  bool more = 3;
  // count is set to the number of keys within the range when requested.
  int64 count = 4;
}
```



首先看这个 header，定义在 etcd 源码的 `etcdserver/etcdserverpb/rpc.proto` 中：

```protobuf
message ResponseHeader {
  // cluster_id is the ID of the cluster which sent the response.
  uint64 cluster_id = 1;
  // member_id is the ID of the member which sent the response.
  uint64 member_id = 2;
  // revision is the key-value store revision when the request was applied.
  // For watch progress responses, the header.revision indicates progress. All future events
  // recieved in this stream are guaranteed to have a higher revision number than the
  // header.revision number.
  int64 revision = 3;
  // raft_term is the raft term when the request was applied.
  uint64 raft_term = 4;
}
```

首先，`cluster_id` 表示一个集群 ID，哪怕你把数据目录全部删掉也不会变。

`member_id` 表示集群中你访问节点的 ID，这个也是不变的。

`revision` 表示集群内，增加或者删除的总次数。这是包含所有 key 的增删改的总次数。

`raft_term` 就表示，etcd 重新选举一次 leader，或者重启一次就会加 1，可以简单理解为重启次数。



然后是 `mvccpb.KeyValue` 这个结构

```protobuf
message KeyValue {
  // key is the key in bytes. An empty key is not allowed.
  bytes key = 1;
  // create_revision is the revision of last creation on this key.
  int64 create_revision = 2;
  // mod_revision is the revision of last modification on this key.
  int64 mod_revision = 3;
  // version is the version of the key. A deletion resets
  // the version to zero and any modification of the key
  // increases its version.
  int64 version = 4;
  // value is the value held by the key, in bytes.
  bytes value = 5;
  // lease is the ID of the lease that attached to key.
  // When the attached lease expires, the key will be deleted.
  // If lease is 0, then no lease is attached to the key.
  int64 lease = 6;
}
```

这里的 `key` 和 `value` 都是 base64 编码的 bytes 型数据。

`version`  就是当前 key 的版本，key 被删除会重置这个值。

`create_revision` 表示当前 key 是在哪一次 `revision` 中被创建的。

`mod_revision` 表示当前 key 是在哪一次 `revision` 中被修改的（注意：创建也是修改）。

