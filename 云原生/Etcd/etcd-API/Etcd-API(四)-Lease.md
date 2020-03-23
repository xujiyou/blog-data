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

Golang 创建租约（有毛病：context canceled"）：

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





## LeaseRevoke

撤销租约

LeaseRevokeRequest

```go
type LeaseRevokeRequest struct {
   // ID is the lease ID to revoke. When the ID is revoked, all associated keys will be deleted.
   ID int64 `protobuf:"varint,1,opt,name=ID,proto3" json:"ID,omitempty"`
}
```

LeaseRevokeResponse

```go
type LeaseRevokeResponse struct {
   Header *ResponseHeader `protobuf:"bytes,1,opt,name=header" json:"header,omitempty"`
}
```

 #### etcdctl 撤销租约：

```bash
$ etcdctl lease revoke 694d7104f3a49704
```

#### Curl 撤销租约：

先找出 lease id：

```bash
$ etcdctl lease list -w json
{"cluster_id":14841639068965178418,"member_id":10276657743932975437,"revision":24,"raft_term":7,"leases":[{"id":7587845213270611733},{"id":7587845213270611735},{"id":7587845213270611737},{"id":7587845213270611739}]}
```

这里说明一下，输出是json格式的才会看到 10 进制的 id，如果像下面这样：

```bash
$ etcdctl lease list        
found 2 leases
694d7104f3a49719
694d7104f3a4971b
```

这样只能获取 16 进制的 id。如果想用到 curl 里面，需要转换格式：

```bash
$ printf %d 0x694d7104f3a49719
7587845213270611737
```

再进行撤销：

```
curl -L http://127.0.0.1:2379/v3/lease/revoke -X POST -d '{"ID": 7587845213270611735}'
```



##### golang 撤销租约：

可以用 10 进制的租约 id，也可以用 16 进制的：

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
		DialTimeout: 20 * time.Second,
	})
	if err == nil {
	  // resp, err := cli.Lease.Revoke(context.TODO(), 0x694d7104f3a49721)
		resp, err := cli.Lease.Revoke(context.TODO(), 7587845213270611733)
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



## LeaseKeepAlive

这是个双向流式的 rpc 方法。

etcdctl 调用：

```bash
$ etcdctl lease keep-alive 694d7104f3a49724
```

golang 调用：

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
      DialTimeout: 20 * time.Second,
   })
   if err == nil {
      ch, err := cli.Lease.KeepAlive(context.TODO(), 0x694d7104f3a49724)
      if err != nil {
         log.Fatalln(err)
      }
      for resp := range ch {
         log.Println(resp)
      }

   } else {
      _ = fmt.Errorf("err: %s", err)
   }
   defer cli.Close()
}
```



## LeaseTimeToLive

LeaseTimeToLiveRequest

```go
type LeaseTimeToLiveRequest struct {
   // ID is the lease ID for the lease.
   ID int64 `protobuf:"varint,1,opt,name=ID,proto3" json:"ID,omitempty"`
   // keys is true to query all the keys attached to this lease.
   Keys bool `protobuf:"varint,2,opt,name=keys,proto3" json:"keys,omitempty"`
}
```

LeaseTimeToLiveResponse

```go
type LeaseTimeToLiveResponse struct {
   Header *ResponseHeader `protobuf:"bytes,1,opt,name=header" json:"header,omitempty"`
   // ID is the lease ID from the keep alive request.
   ID int64 `protobuf:"varint,2,opt,name=ID,proto3" json:"ID,omitempty"`
   // TTL is the remaining TTL in seconds for the lease; the lease will expire in under TTL+1 seconds.
   TTL int64 `protobuf:"varint,3,opt,name=TTL,proto3" json:"TTL,omitempty"`
   // GrantedTTL is the initial granted time in seconds upon lease creation/renewal.
   GrantedTTL int64 `protobuf:"varint,4,opt,name=grantedTTL,proto3" json:"grantedTTL,omitempty"`
   // Keys is the list of keys attached to this lease.
   Keys [][]byte `protobuf:"bytes,5,rep,name=keys" json:"keys,omitempty"`
}
```

etcdctl 访问：

```bash
$ etcdctl lease timetolive 694d7104f3a49724
lease 694d7104f3a49724 granted with TTL(300s), remaining(265s)
```

curl 访问：

```bash
$ curl http://127.0.0.1:2379/v3/lease/timetolive -X POST -d '{"ID": 7587845213270611748, "keys": true}'
```

golang 没有这个方法。



## LeaseLeases

没有请求信息。

LeaseTimeToLiveResponse

```go
type LeaseTimeToLiveResponse struct {
   Header *ResponseHeader `protobuf:"bytes,1,opt,name=header" json:"header,omitempty"`
   // ID is the lease ID from the keep alive request.
   ID int64 `protobuf:"varint,2,opt,name=ID,proto3" json:"ID,omitempty"`
   // TTL is the remaining TTL in seconds for the lease; the lease will expire in under TTL+1 seconds.
   TTL int64 `protobuf:"varint,3,opt,name=TTL,proto3" json:"TTL,omitempty"`
   // GrantedTTL is the initial granted time in seconds upon lease creation/renewal.
   GrantedTTL int64 `protobuf:"varint,4,opt,name=grantedTTL,proto3" json:"grantedTTL,omitempty"`
   // Keys is the list of keys attached to this lease.
   Keys [][]byte `protobuf:"bytes,5,rep,name=keys" json:"keys,omitempty"`
}
```

etcdctl 访问：

```bash
$ etcdctl lease list
```

curl 访问：

```
$ curl http://127.0.0.1:2379/v3/lease/leases -X POST
```

golang 也没有这个方法。





















