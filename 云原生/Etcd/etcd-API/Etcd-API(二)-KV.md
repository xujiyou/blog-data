# Etcd API (二) - KV

下面先来看 KV 服务，其中定义了最基本最常用的 RPC 方法：

```protobuf
service KV {
  // Range gets the keys in the range from the key-value store.
  rpc Range(RangeRequest) returns (RangeResponse) {
      option (google.api.http) = {
        post: "/v3/kv/range"
        body: "*"
    };
  }

  // Put puts the given key into the key-value store.
  // A put request increments the revision of the key-value store
  // and generates one event in the event history.
  rpc Put(PutRequest) returns (PutResponse) {
      option (google.api.http) = {
        post: "/v3/kv/put"
        body: "*"
    };
  }

  // DeleteRange deletes the given range from the key-value store.
  // A delete request increments the revision of the key-value store
  // and generates a delete event in the event history for every deleted key.
  rpc DeleteRange(DeleteRangeRequest) returns (DeleteRangeResponse) {
      option (google.api.http) = {
        post: "/v3/kv/deleterange"
        body: "*"
    };
  }

  // Txn processes multiple requests in a single transaction.
  // A txn request increments the revision of the key-value store
  // and generates events with the same revision for every completed request.
  // It is not allowed to modify the same key several times within one txn.
  rpc Txn(TxnRequest) returns (TxnResponse) {
      option (google.api.http) = {
        post: "/v3/kv/txn"
        body: "*"
    };
  }

  // Compact compacts the event history in the etcd key-value store. The key-value
  // store should be periodically compacted or the event history will continue to grow
  // indefinitely.
  // 键值存储区应定期压缩，否则事件历史记录将无限期地增长。
  rpc Compact(CompactionRequest) returns (CompactionResponse) {
      option (google.api.http) = {
        post: "/v3/kv/compaction"
        body: "*"
    };
  }
}
```

可以看到一个 5 个方法。除此之外，还可以看到 URL 路径及方法

在这个 `rpc.proto` 文件的相同目录下，有一个 `rpc.pb.go` 文件，这个文件是 protoc 生成的，里面有一个 `KVServer` 接口：

```go
type KVServer interface {
  Range(context.Context, *RangeRequest) (*RangeResponse, error)
  Put(context.Context, *PutRequest) (*PutResponse, error)
  DeleteRange(context.Context, *DeleteRangeRequest) (*DeleteRangeResponse, error)
  Txn(context.Context, *TxnRequest) (*TxnResponse, error)
  Compact(context.Context, *CompactionRequest) (*CompactionResponse, error)
}
```

在 Goland 里面按住 ctrl + h 可以查看接口的实现。

这个接口的实现在 `etcdserver/api/v3rpc/key.go` 目录下，有一个名为 `kvServer` 的结构体。

这个结构体提供了 5 个方法的实现。具体实现就不看了，太多了。

只是学习 API 的话，只看各种 Request 和 Response 结构体就可以了。



## Range

RangeRequest:

```go
type RangeRequest struct {
  // 如果 range_end 没有设置，这次请求仅仅就是查找 key。
	Key []byte `protobuf:"bytes,1,opt,name=key,proto3" json:"key,omitempty"`
  // range_end 是一个左闭右开的区间，如：[key, range_end)
  // 如果 range_end 是 '\0'，则会查找所有大于等于 key 的键(keys >= key)
	// 如果 range_end 是 key + 1 (比如"aa"+1 == "ab", "a\xff"+1 == "b"),这样会请求获取所有以key开头的键。
  // 如果 key 和 range_end 都是 '\0'，则这次请求会列出所有的键值对。
	RangeEnd []byte `protobuf:"bytes,2,opt,name=range_end,json=rangeEnd,proto3" json:"range_end,omitempty"`
  // 限制请求返回的 key 的数量，当设置为 0 时，表示没有限制。
	Limit int64 `protobuf:"varint,3,opt,name=limit,proto3" json:"limit,omitempty"`
  // revision 是这次查询的数据保存的时间点
  // 如果 revision 小于或等于 0，这次查询会查询最新的。
  // 如果 revision 记录的是删除操作，则会响应 ErrCompacted
	Revision int64 `protobuf:"varint,4,opt,name=revision,proto3" json:"revision,omitempty"`
	// 排序方式，RangeRequest_NONE、RangeRequest_ASCEND、RangeRequest_DESCEND
	SortOrder RangeRequest_SortOrder `protobuf:"varint,5,opt,name=sort_order,json=sortOrder,proto3,enum=etcdserverpb.RangeRequest_SortOrder" json:"sort_order,omitempty"`
	// 排序目标，RangeRequest_KEY、RangeRequest_VERSION、RangeRequest_CREATE、RangeRequest_MOD、RangeRequest_VALUE
	SortTarget RangeRequest_SortTarget `protobuf:"varint,6,opt,name=sort_target,json=sortTarget,proto3,enum=etcdserverpb.RangeRequest_SortTarget" json:"sort_target,omitempty"`
  // serializable 设置了这次请求会用于序列化本地成员的读取
  // Range 请求默认是序列化的。序列化请求具有更高的延迟和更低的吞吐量
	// 为了获得更好的性能，为了交换可能的陈旧读取，在本地服务可序列化范围请求，而无需与集群中的其他节点达成共识。
	Serializable bool `protobuf:"varint,7,opt,name=serializable,proto3" json:"serializable,omitempty"`
	// 只获取 key，不获取 value
	KeysOnly bool `protobuf:"varint,8,opt,name=keys_only,json=keysOnly,proto3" json:"keys_only,omitempty"`
	// 只获取数量
	CountOnly bool `protobuf:"varint,9,opt,name=count_only,json=countOnly,proto3" json:"count_only,omitempty"`
	// 最小的 ModRevision
	MinModRevision int64 `protobuf:"varint,10,opt,name=min_mod_revision,json=minModRevision,proto3" json:"min_mod_revision,omitempty"`
	// 最大的ModRevision
	MaxModRevision int64 `protobuf:"varint,11,opt,name=max_mod_revision,json=maxModRevision,proto3" json:"max_mod_revision,omitempty"`
	// 最小的CreateRevision
	MinCreateRevision int64 `protobuf:"varint,12,opt,name=min_create_revision,json=minCreateRevision,proto3" json:"min_create_revision,omitempty"`
	// 最大的CreateRevision
	MaxCreateRevision int64 `protobuf:"varint,13,opt,name=max_create_revision,json=maxCreateRevision,proto3" json:"max_create_revision,omitempty"`
}
```

RangeResponse：

```go
type RangeResponse struct {
   Header *ResponseHeader `protobuf:"bytes,1,opt,name=header" json:"header,omitempty"`
   // kvs is the list of key-value pairs matched by the range request.
   // kvs is empty when count is requested.
   Kvs []*mvccpb.KeyValue `protobuf:"bytes,2,rep,name=kvs" json:"kvs,omitempty"`
   // more indicates if there are more keys to return in the requested range.
   More bool `protobuf:"varint,3,opt,name=more,proto3" json:"more,omitempty"`
   // count is set to the number of keys within the range when requested.
   Count int64 `protobuf:"varint,4,opt,name=count,proto3" json:"count,omitempty"`
}
```

这个响应在  [etcd中的各种版本.md](../etcd中的各种版本.md) 中已经学习了。



先通过 etcdctl 访问这个 API

首先查看帮助：

```
$ etcdctl get -h
NAME:
        get - Gets the key or a range of keys

USAGE:
        etcdctl get [options] <key> [range_end] [flags]

OPTIONS:
      --consistency="l"                 Linearizable(l) or Serializable(s)
      --from-key[=false]                Get keys that are greater than or equal to the given key using byte compare
  -h, --help[=false]                    help for get
      --keys-only[=false]               Get only the keys
      --limit=0                         Maximum number of results
      --order=""                        Order of results; ASCEND or DESCEND (ASCEND by default)
      --prefix[=false]                  Get keys with matching prefix
      --print-value-only[=false]        Only write values when using the "simple" output format
      --rev=0                           Specify the kv revision
      --sort-by=""                      Sort target; CREATE, KEY, MODIFY, VALUE, or VERSION
```

看这些参数就是对应的上边的 RangeRequest 结构体中的字段。

--from-key 表示 使用字节大于或等于给定 BASE64 字符串的 range_end。这个参数用到下面就可以获取全部 key

获取全部key（避免过多，限制数量为10）：

```bash
$ etcdctl get "" --from-key --limit=10
```

返回json：

```bash
$ etcdctl get "" --from-key --limit=10 -w json | json_pp
{
   "kvs" : [
      {
         "value" : "YmFyNg==",
         "mod_revision" : 15,
         "create_revision" : 15,
         "version" : 1,
         "key" : "Zm9v"
      },
      {
         "version" : 1,
         "create_revision" : 12,
         "key" : "Zm9vMQ==",
         "mod_revision" : 12,
         "value" : "YmFyNA=="
      }
   ],
   "count" : 2,
   "header" : {
      "raft_term" : 4,
      "revision" : 15,
      "member_id" : 10276657743932975437,
      "cluster_id" : 14841639068965178418
   }
}
```

使用 curl 访问：

```bash
$ curl -L http://localhost:2379/v3/kv/range -X POST -d '{"key": "Zm9v", "range_end": "Zm9w", "limit": 10}' | json_pp
{
   "header" : {
      "raft_term" : "4",
      "revision" : "15",
      "member_id" : "10276657743932975437",
      "cluster_id" : "14841639068965178418"
   },
   "count" : "2",
   "kvs" : [
      {
         "mod_revision" : "15",
         "value" : "YmFyNg==",
         "create_revision" : "15",
         "key" : "Zm9v",
         "version" : "1"
      },
      {
         "value" : "YmFyNA==",
         "mod_revision" : "12",
         "key" : "Zm9vMQ==",
         "create_revision" : "12",
         "version" : "1"
      }
   ]
}
```

curl 有个弊端，就是需要自己生成 BASE64 的数据。看到上边填充的 key 和 range_end 这两个字段，是不是跟之前学到的一样那。

然后再用 golang 测试一下：

具体写法在  [etcd-golang客户端.md](../etcd-golang客户端.md) 都介绍了，下面仅仅列出差异部分：

```go
    resp,_ := cli.KV.Get(context.TODO(), "0", clientv3.WithLimit(10), clientv3.WithFromKey())
		for i := range resp.Kvs {
			fmt.Printf("key: %s; resp value: %s\n", resp.Kvs[i].Key, resp.Kvs[i].Value)
		}
```



