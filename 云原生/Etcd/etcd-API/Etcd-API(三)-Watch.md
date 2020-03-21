# Etcd API (三) - Watch

先查看 proto 文件关于 Watch 的内容：

```protobuf
service Watch {
  // Watch watches for events happening or that have happened. Both input and output
  // are streams; the input stream is for creating and canceling watchers and the output
  // stream sends events. One watch RPC can watch on multiple key ranges, streaming events
  // for several watches at once. The entire event history can be watched starting from the
  // last compaction revision.
  rpc Watch(stream WatchRequest) returns (stream WatchResponse) {
      option (google.api.http) = {
        post: "/v3/watch"
        body: "*"
    };
  }
}
```

只有一个 rpc 方法，并且这个方法是一个双向流，有关双向流的学习，可以看： [gRPC流式传输.md](../../gRPC/gRPC流式传输.md) 。

查看接口：

```go
type WatchServer interface {
	// Watch watches for events happening or that have happened. Both input and output
	// are streams; the input stream is for creating and canceling watchers and the output
	// stream sends events. One watch RPC can watch on multiple key ranges, streaming events
	// for several watches at once. The entire event history can be watched starting from the
	// last compaction revision.
	Watch(Watch_WatchServer) error
}
```

Watch_WatchServer 也是一个接口，查看这个接口的定义：

```go
type Watch_WatchServer interface {
	Send(*WatchResponse) error
	Recv() (*WatchRequest, error)
	grpc.ServerStream
}
```

很正常的一个流式接口的定义，下面看 WatchResponse：

```go
type WatchResponse struct {
   Header *ResponseHeader `protobuf:"bytes,1,opt,name=header" json:"header,omitempty"`
   // watch_id is the ID of the watcher that corresponds to the response.
   WatchId int64 `protobuf:"varint,2,opt,name=watch_id,json=watchId,proto3" json:"watch_id,omitempty"`
   // 如果为true，则表示 watch 刚刚创建
   // created is set to true if the response is for a create watch request.
   // The client should record the watch_id and expect to receive events for
   // the created watcher from the same stream.
   // All events sent to the created watcher will attach with the same watch_id.
   Created bool `protobuf:"varint,3,opt,name=created,proto3" json:"created,omitempty"`
   // 如果为 true，则表明 watch 取消了
   // canceled is set to true if the response is for a cancel watch request.
   // No further events will be sent to the canceled watcher.
   Canceled bool `protobuf:"varint,4,opt,name=canceled,proto3" json:"canceled,omitempty"`
  // 如果 watch 的是一个  compacted 过的 revision 就会设置这个字段
  // compact_revision is set to the minimum index if a watcher tries to watch
   // at a compacted index.
   //
   // This happens when creating a watcher at a compacted revision or the watcher cannot
   // catch up with the progress of the key-value store.
   //
   // The client should treat the watcher as canceled and should not try to create any
   // watcher with the same start_revision again.
   CompactRevision int64 `protobuf:"varint,5,opt,name=compact_revision,json=compactRevision,proto3" json:"compact_revision,omitempty"`
   // 取消原因
   CancelReason string `protobuf:"bytes,6,opt,name=cancel_reason,json=cancelReason,proto3" json:"cancel_reason,omitempty"`
   // 如果为 true，则大的响应会分隔为多个段
   Fragment bool            `protobuf:"varint,7,opt,name=fragment,proto3" json:"fragment,omitempty"`
   Events   []*mvccpb.Event `protobuf:"bytes,11,rep,name=events" json:"events,omitempty"`
}
```

WatchRequest:

```go
type WatchRequest struct {
   // request_union is a request to either create a new watcher or cancel an existing watcher.
   //
   // Types that are valid to be assigned to RequestUnion:
   // *WatchRequest_CreateRequest
   // *WatchRequest_CancelRequest
   // *WatchRequest_ProgressRequest
   RequestUnion isWatchRequest_RequestUnion `protobuf_oneof:"request_union"`
}
```



下面先使用 etcdctl 来验证：

```bash
$ etcdctl watch foo
```

然后在另一个命令行窗口执行：

```bash
$ etcdctl put foo bar7
```



curl 无法验证，因为 RequestUnion 不支持 json 字段，下面用 golang 验证。

```go
package main

import (
   "context"
   "fmt"
   "go.etcd.io/etcd/clientv3"
   "log"
   "time"
)

func main() {

   cli, err := clientv3.New(clientv3.Config{
      Endpoints:   []string{"127.0.0.1:2379"},
      RetryDialer: nil,
      DialTimeout: 5 * time.Second,
   })
   if err == nil {
      for resp := range cli.Watcher.Watch(context.TODO(), "foo", clientv3.WithPrefix()) {
         log.Println("%s", resp)
      }
   } else {
      _ = fmt.Errorf("err: %s", err)
   }
   defer cli.Close()
}
```

启动程序，然后在命令行执行 `etcdctl put foo bar8`，看看程序输出了什么。











