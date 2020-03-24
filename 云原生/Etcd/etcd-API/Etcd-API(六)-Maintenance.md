# Etcd API (六) - Maintenance

 Maintenance 是保养的意思。

```protobuf
service Maintenance {
  // Alarm activates, deactivates, and queries alarms regarding cluster health.
  rpc Alarm(AlarmRequest) returns (AlarmResponse) {
      option (google.api.http) = {
        post: "/v3/maintenance/alarm"
        body: "*"
    };
  }

  // Status gets the status of the member.
  rpc Status(StatusRequest) returns (StatusResponse) {
      option (google.api.http) = {
        post: "/v3/maintenance/status"
        body: "*"
    };
  }

  // Defragment defragments a member's backend database to recover storage space.
  rpc Defragment(DefragmentRequest) returns (DefragmentResponse) {
      option (google.api.http) = {
        post: "/v3/maintenance/defragment"
        body: "*"
    };
  }

  // Hash computes the hash of whole backend keyspace,
  // including key, lease, and other buckets in storage.
  // This is designed for testing ONLY!
  // Do not rely on this in production with ongoing transactions,
  // since Hash operation does not hold MVCC locks.
  // Use "HashKV" API instead for "key" bucket consistency checks.
  rpc Hash(HashRequest) returns (HashResponse) {
      option (google.api.http) = {
        post: "/v3/maintenance/hash"
        body: "*"
    };
  }

  // HashKV computes the hash of all MVCC keys up to a given revision.
  // It only iterates "key" bucket in backend storage.
  rpc HashKV(HashKVRequest) returns (HashKVResponse) {
      option (google.api.http) = {
        post: "/v3/maintenance/hash"
        body: "*"
    };
  }

  // Snapshot sends a snapshot of the entire backend from a member over a stream to a client.
  rpc Snapshot(SnapshotRequest) returns (stream SnapshotResponse) {
      option (google.api.http) = {
        post: "/v3/maintenance/snapshot"
        body: "*"
    };
  }

  // MoveLeader requests current leader node to transfer its leadership to transferee.
  rpc MoveLeader(MoveLeaderRequest) returns (MoveLeaderResponse) {
      option (google.api.http) = {
        post: "/v3/maintenance/transfer-leadership"
        body: "*"
    };
  }
}
```

golang 接口：

```go
type MaintenanceServer interface {
	Alarm(context.Context, *AlarmRequest) (*AlarmResponse, error)
	Status(context.Context, *StatusRequest) (*StatusResponse, error)
	Defragment(context.Context, *DefragmentRequest) (*DefragmentResponse, error)
	Hash(context.Context, *HashRequest) (*HashResponse, error)
	HashKV(context.Context, *HashKVRequest) (*HashKVResponse, error)
	Snapshot(*SnapshotRequest, Maintenance_SnapshotServer) error
	MoveLeader(context.Context, *MoveLeaderRequest) (*MoveLeaderResponse, error)
}
```

7 个方法



## Alarm

报警

AlarmRequest

```go
type AlarmRequest struct {
   // action is the kind of alarm request to issue. The action
   // may GET alarm statuses, ACTIVATE an alarm, or DEACTIVATE a
   // raised alarm.
   Action AlarmRequest_AlarmAction `protobuf:"varint,1,opt,name=action,proto3,enum=etcdserverpb.AlarmRequest_AlarmAction" json:"action,omitempty"`
   // memberID is the ID of the member associated with the alarm. If memberID is 0, the
   // alarm request covers all members.
   MemberID uint64 `protobuf:"varint,2,opt,name=memberID,proto3" json:"memberID,omitempty"`
   // alarm is the type of alarm to consider for this request.
   Alarm AlarmType `protobuf:"varint,3,opt,name=alarm,proto3,enum=etcdserverpb.AlarmType" json:"alarm,omitempty"`
}
```

AlarmResponse

```go
type AlarmResponse struct {
   Header *ResponseHeader `protobuf:"bytes,1,opt,name=header" json:"header,omitempty"`
   // alarms is a list of alarms associated with the alarm request.
   Alarms []*AlarmMember `protobuf:"bytes,2,rep,name=alarms" json:"alarms,omitempty"`
}
```

因为这个报警是 etcd 内部的报警，所有通过 etcdctl 是无法创建报警的。golang 也无法创建，curl 更不用试了。

当内部创建了报警后，可以通过 `etcdctl alarm list` 来查看



##Status

没有请求数据

StatusResponse：

```go
type StatusResponse struct {
   Header *ResponseHeader `protobuf:"bytes,1,opt,name=header" json:"header,omitempty"`
   // version is the cluster protocol version used by the responding member.
   Version string `protobuf:"bytes,2,opt,name=version,proto3" json:"version,omitempty"`
   // dbSize is the size of the backend database physically allocated, in bytes, of the responding member.
   DbSize int64 `protobuf:"varint,3,opt,name=dbSize,proto3" json:"dbSize,omitempty"`
   // leader is the member ID which the responding member believes is the current leader.
   Leader uint64 `protobuf:"varint,4,opt,name=leader,proto3" json:"leader,omitempty"`
   // raftIndex is the current raft committed index of the responding member.
   RaftIndex uint64 `protobuf:"varint,5,opt,name=raftIndex,proto3" json:"raftIndex,omitempty"`
   // raftTerm is the current raft term of the responding member.
   RaftTerm uint64 `protobuf:"varint,6,opt,name=raftTerm,proto3" json:"raftTerm,omitempty"`
   // raftAppliedIndex is the current raft applied index of the responding member.
   RaftAppliedIndex uint64 `protobuf:"varint,7,opt,name=raftAppliedIndex,proto3" json:"raftAppliedIndex,omitempty"`
   // errors contains alarm/health information and status.
   Errors []string `protobuf:"bytes,8,rep,name=errors" json:"errors,omitempty"`
   // dbSizeInUse is the size of the backend database logically in use, in bytes, of the responding member.
   DbSizeInUse int64 `protobuf:"varint,9,opt,name=dbSizeInUse,proto3" json:"dbSizeInUse,omitempty"`
   // isLearner indicates if the member is raft learner.
   IsLearner bool `protobuf:"varint,10,opt,name=isLearner,proto3" json:"isLearner,omitempty"`
}
```

etcdctl 访问：

```bash
$ etcdctl endpoint status -w table
```

curl 访问：

```bash
$ curl -X POST http://127.0.0.1:2379/v3/maintenance/status | json_reformat 
```



## Defragment

碎片整理

没有请求数据，

DefragmentResponse

```go
type DefragmentResponse struct {
   Header *ResponseHeader `protobuf:"bytes,1,opt,name=header" json:"header,omitempty"`
}
```

Etcdctl 访问：

```bash
$ etcdctl defrag
Finished defragmenting etcd member[127.0.0.1:2379]
```

curl 访问：

```bash
$ curl -X POST http://127.0.0.1:2379/v3/maintenance/defragment | json_reformat
```

golang 访问 ：

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
		resp, err := cli.Maintenance.Defragment(context.TODO(), "127.0.0.1:2379")
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



## Hash

哈希计算整个后端键空间的哈希，
  包括钥匙，租赁和其他存储桶。
这仅用于测试！
不要在正在进行的交易中依赖于此，
因为哈希操作不持有MVCC锁。
请改用“ HashKV” API进行“密钥”存储桶一致性检查。



## HashKV

HashKV计算直到给定版本的所有MVCC密钥的哈希。
它仅迭代后端存储中的“密钥”存储区。

HashKVRequest

```go
type HashKVRequest struct {
   // revision is the key-value store revision for the hash operation.
   Revision int64 `protobuf:"varint,1,opt,name=revision,proto3" json:"revision,omitempty"`
}
```

HashKVResponse

```go
type HashKVResponse struct {
   Header *ResponseHeader `protobuf:"bytes,1,opt,name=header" json:"header,omitempty"`
   // hash is the hash value computed from the responding member's MVCC keys up to a given revision.
   Hash uint32 `protobuf:"varint,2,opt,name=hash,proto3" json:"hash,omitempty"`
   // compact_revision is the compacted revision of key-value store when hash begins.
   CompactRevision int64 `protobuf:"varint,3,opt,name=compact_revision,json=compactRevision,proto3" json:"compact_revision,omitempty"`
}
```

etcdctl 访问:

```bash
$ etcdctl endpoint hashkv -w json | json_reformat 
[
    {
        "Endpoint": "127.0.0.1:2379",
        "HashKV": {
            "header": {
                "cluster_id": 14841639068965178418,
                "member_id": 10276657743932975437,
                "revision": 24,
                "raft_term": 8
            },
            "hash": 3526116449,
            "compact_revision": 18
        }
    }
]
```

Curl 访问：

```bash
$ curl -X POST http://127.0.0.1:2379/v3/maintenance/hash | json_reformat
```

golang 客户端库不支持访问。



## Snapshot

这是一个服务端流的RPC。

这里学习下快照的用法：

```bash
$ # 保存快照
$ etcdctl snapshot save a.db
$ # 查看快照状态
$ etcdctl snapshot status a.db
$ # 恢复 数据到目录
$ etcdctl snapshot restore a.db --data-dir=./a
```



## MoveLeader

MoveLeaderRequest

```go
type MoveLeaderRequest struct {
   // targetID is the node ID for the new leader.
   TargetID uint64 `protobuf:"varint,1,opt,name=targetID,proto3" json:"targetID,omitempty"`
}
```

MoveLeaderResponse

```go
type MoveLeaderResponse struct {
   Header *ResponseHeader `protobuf:"bytes,1,opt,name=header" json:"header,omitempty"`
}
```

etcdctl 示例：

```bash
$ etcdctl member list
8e9e05c52164694d, started, default, http://localhost:2380, http://localhost:2379, false
$ etcdctl move-leader 8e9e05c52164694d
Leadership transferred from 8e9e05c52164694d to 8e9e05c52164694d
```

curl 访问：

```bash
$ curl http://127.0.0.1:2379/v3/maintenance/transfer-leadership -X POST -d '{"targetID": 9223372036854775807}'
```













