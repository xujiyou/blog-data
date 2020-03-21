# Etcd API (四) - Lease

 先来看 proto 文件中关于 Lease 的定义：

```go
service Lease {
  // LeaseGrant创建一个租约，如果服务器在给定的生存期内没有收到keepAlive，则该租约将过期。附加到租约的所有 key 都将过期，如果租约过期，则会删除这些 key。每个过期的 key 在事件历史记录中生成一个删除事件。
  rpc LeaseGrant(LeaseGrantRequest) returns (LeaseGrantResponse) {
      option (google.api.http) = {
        post: "/v3/lease/grant"
        body: "*"
    };
  }

  // 吊销 lease 。附加到 lease 的所有 key 都将过期并被删除。
  rpc LeaseRevoke(LeaseRevokeRequest) returns (LeaseRevokeResponse) {
      option (google.api.http) = {
        post: "/v3/lease/revoke"
        body: "*"
        additional_bindings {
            post: "/v3/kv/lease/revoke"
            body: "*"
        }
    };
  }

  // LeaseKeepAlive 通过将来自客户端的keep alive请求流式传输到服务器并将来自服务器的keep alive响应流式传输到客户端来保持租约的活动。
  rpc LeaseKeepAlive(stream LeaseKeepAliveRequest) returns (stream LeaseKeepAliveResponse) {
      option (google.api.http) = {
        post: "/v3/lease/keepalive"
        body: "*"
    };
  }

  // LeaseTimeToLive 检索 lease 信息。
  rpc LeaseTimeToLive(LeaseTimeToLiveRequest) returns (LeaseTimeToLiveResponse) {
      option (google.api.http) = {
        post: "/v3/lease/timetolive"
        body: "*"
        additional_bindings {
            post: "/v3/kv/lease/timetolive"
            body: "*"
        }
    };
  }

  // LeaseLeases 列出所有的租约。
  rpc LeaseLeases(LeaseLeasesRequest) returns (LeaseLeasesResponse) {
      option (google.api.http) = {
        post: "/v3/lease/leases"
        body: "*"
        additional_bindings {
            post: "/v3/kv/lease/leases"
            body: "*"
        }
    };
  }
}
```

共 5 个 rpc 方法，实际接口：

```go
type LeaseServer interface {
	LeaseGrant(context.Context, *LeaseGrantRequest) (*LeaseGrantResponse, error)
	LeaseRevoke(context.Context, *LeaseRevokeRequest) (*LeaseRevokeResponse, error)
	LeaseKeepAlive(Lease_LeaseKeepAliveServer) error
	LeaseTimeToLive(context.Context, *LeaseTimeToLiveRequest) (*LeaseTimeToLiveResponse, error)
	LeaseLeases(context.Context, *LeaseLeasesRequest) (*LeaseLeasesResponse, error)
}
```



## LeaseGrant

LeaseGrantRequest

```go
type LeaseGrantRequest struct {
   // TTL是以秒为单位的建议生存时间。过期租约将返回-1。
   TTL int64 `protobuf:"varint,1,opt,name=TTL,proto3" json:"TTL,omitempty"`
   // ID是租约的请求ID。如果ID设置为0，则出租人选择ID。
   ID int64 `protobuf:"varint,2,opt,name=ID,proto3" json:"ID,omitempty"`
}
```

LeaseGrantResponse

```go
type LeaseGrantResponse struct {
   Header *ResponseHeader `protobuf:"bytes,1,opt,name=header" json:"header,omitempty"`
   // ID is the lease ID for the granted lease.
   ID int64 `protobuf:"varint,2,opt,name=ID,proto3" json:"ID,omitempty"`
   // TTL is the server chosen lease time-to-live in seconds.
   TTL   int64  `protobuf:"varint,3,opt,name=TTL,proto3" json:"TTL,omitempty"`
   Error string `protobuf:"bytes,4,opt,name=error,proto3" json:"error,omitempty"`
}
```

etchctl创建 lease：

```bash
$ etcdctl lease grant 300
lease 694d70faca81d808 granted with TTL(300s)
```

curl 创建租约

```bash
$ curl -L http://localhost:2379/v3/lease/grant -X POST -d '{"TTL": 300, "ID": 0}'
{"header":{"cluster_id":"14841639068965178418","member_id":"10276657743932975437","revision":"23","raft_term":"6"},"ID":"7587845169630795786","TTL":"300"}
```

Golang 创建租约：

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
		DialTimeout: 20 * time.Second,
	})
	if err == nil {

		resp, err := cli.Lease.Create(context.TODO(), 300)
		if err == nil {
			log.Println("%s", resp)
		} else {
			log.Fatalln("%v", err)
		}

	} else {
		_ = fmt.Errorf("err: %s", err)
	}
	defer cli.Close()
}

```











 



