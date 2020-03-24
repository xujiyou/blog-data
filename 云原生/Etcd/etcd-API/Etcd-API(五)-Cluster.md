# Etcd API (五) - Cluster

 rpc service 定义

```protobuf
service Cluster {
  // MemberAdd adds a member into the cluster.
  rpc MemberAdd(MemberAddRequest) returns (MemberAddResponse) {
      option (google.api.http) = {
        post: "/v3/cluster/member/add"
        body: "*"
    };
  }

  // MemberRemove removes an existing member from the cluster.
  rpc MemberRemove(MemberRemoveRequest) returns (MemberRemoveResponse) {
      option (google.api.http) = {
        post: "/v3/cluster/member/remove"
        body: "*"
    };
  }

  // MemberUpdate updates the member configuration.
  rpc MemberUpdate(MemberUpdateRequest) returns (MemberUpdateResponse) {
      option (google.api.http) = {
        post: "/v3/cluster/member/update"
        body: "*"
    };
  }

  // MemberList lists all the members in the cluster.
  rpc MemberList(MemberListRequest) returns (MemberListResponse) {
      option (google.api.http) = {
        post: "/v3/cluster/member/list"
        body: "*"
    };
  }

  // MemberPromote promotes a member from raft learner (non-voting) to raft voting member.
  rpc MemberPromote(MemberPromoteRequest) returns (MemberPromoteResponse) {
      option (google.api.http) = {
        post: "/v3/cluster/member/promote"
        body: "*"
    };
  }
}
```

具体接口：

```go
type ClusterServer interface {
   MemberAdd(context.Context, *MemberAddRequest) (*MemberAddResponse, error)
   MemberRemove(context.Context, *MemberRemoveRequest) (*MemberRemoveResponse, error)
   MemberUpdate(context.Context, *MemberUpdateRequest) (*MemberUpdateResponse, error)
   MemberList(context.Context, *MemberListRequest) (*MemberListResponse, error)
   MemberPromote(context.Context, *MemberPromoteRequest) (*MemberPromoteResponse, error)
}
```



## MemberAdd

可以预见，etcd 通过动态方式创建集群的时候就是通过这些接口来实现的。

MemberAddRequest

```go
type MemberAddRequest struct {
   // peerURLs is the list of URLs the added member will use to communicate with the cluster.
   PeerURLs []string `protobuf:"bytes,1,rep,name=peerURLs" json:"peerURLs,omitempty"`
   // isLearner indicates if the added member is raft learner.
   IsLearner bool `protobuf:"varint,2,opt,name=isLearner,proto3" json:"isLearner,omitempty"`
}
```

MemberAddResponse

```go
type MemberAddResponse struct {
   Header *ResponseHeader `protobuf:"bytes,1,opt,name=header" json:"header,omitempty"`
   // member is the member information for the added member.
   Member *Member `protobuf:"bytes,2,opt,name=member" json:"member,omitempty"`
   // members is a list of all members after adding the new member.
   Members []*Member `protobuf:"bytes,3,rep,name=members" json:"members,omitempty"`
}
```

etcdctl 示例：

```bash
$ etcdctl member add
```



## MemberList

没有请求数据

MemberListResponse：

```go
type MemberListResponse struct {
   Header *ResponseHeader `protobuf:"bytes,1,opt,name=header" json:"header,omitempty"`
   // members is a list of all members associated with the cluster.
   Members []*Member `protobuf:"bytes,2,rep,name=members" json:"members,omitempty"`
}
```

Etcdctl 访问：

```bash
$ etcdctl member list
```

curl 访问：

```bash
$ curl http://127.0.0.1:2379/v3/cluster/member/list -X POST
```

golang 访问(有毛病，估计是为了安全，有认证啥的)：

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
		resp, err := cli.Cluster.MemberList(context.TODO())
		if err != nil {
			log.Fatalln(err)
		}
		log.Println(resp)

	} else {
		_ = fmt.Errorf("err: %s", err)
	}
	defer cli.Close()
}

```











