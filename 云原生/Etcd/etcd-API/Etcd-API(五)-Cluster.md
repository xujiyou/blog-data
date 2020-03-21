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

