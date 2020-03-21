# Etcd API (四) - Lease

 先来看 proto 文件中关于 Lease 的定义：

```go
service Lease {
  // LeaseGrant creates a lease which expires if the server does not receive a keepAlive
  // within a given time to live period. All keys attached to the lease will be expired and
  // deleted if the lease expires. Each expired key generates a delete event in the event history.
  rpc LeaseGrant(LeaseGrantRequest) returns (LeaseGrantResponse) {
      option (google.api.http) = {
        post: "/v3/lease/grant"
        body: "*"
    };
  }

  // LeaseRevoke revokes a lease. All keys attached to the lease will expire and be deleted.
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

  // LeaseKeepAlive keeps the lease alive by streaming keep alive requests from the client
  // to the server and streaming keep alive responses from the server to the client.
  rpc LeaseKeepAlive(stream LeaseKeepAliveRequest) returns (stream LeaseKeepAliveResponse) {
      option (google.api.http) = {
        post: "/v3/lease/keepalive"
        body: "*"
    };
  }

  // LeaseTimeToLive retrieves lease information.
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

  // LeaseLeases lists all existing leases.
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

