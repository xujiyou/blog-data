# Kubernetes API 学习(十六) - coordination

coordination 是 协调的意思。

```bash
$ kubectl api-resources --api-group=coordination.k8s.io
NAME     SHORTNAMES   APIGROUP              NAMESPACED   KIND
leases                coordination.k8s.io   true         Lease
```

可以查看 [https://kuboard.cn/learning/k8s-bg/architecture/nodes-mgmt.html#%E8%8A%82%E7%82%B9%E6%8E%A7%E5%88%B6%E5%99%A8%EF%BC%88node-controller%EF%BC%89](https://kuboard.cn/learning/k8s-bg/architecture/nodes-mgmt.html#节点控制器（node-controller）) 这篇文章的 Lease 部分。



源码在 API 源码包的 coordination 包中。

对象结构：

```go
type Lease struct {
	metav1.TypeMeta `json:",inline"`
	// More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
	// +optional
	metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

	// Specification of the Lease.
	// More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
	// +optional
	Spec LeaseSpec `json:"spec,omitempty" protobuf:"bytes,2,opt,name=spec"`
}
```

LeaseSpec:

```go
type LeaseSpec struct {
   // holderIdentity包含当前 Lease 的所有者的身份。
   // +optional
   HolderIdentity *string `json:"holderIdentity,omitempty" protobuf:"bytes,1,opt,name=holderIdentity"`
   // leaseDurationSeconds是 Lease 候选人需要等待强制强制获取 Lease 的持续时间。 这是针对上次观察到的RenewTime时间的度量。
   // +optional
   LeaseDurationSeconds *int32 `json:"leaseDurationSeconds,omitempty" protobuf:"varint,2,opt,name=leaseDurationSeconds"`
   // acquisitionTime是获取当前 Lease 的时间。
   // +optional
   AcquireTime *metav1.MicroTime `json:"acquireTime,omitempty" protobuf:"bytes,3,opt,name=acquireTime"`
   // renewTime是租赁的当前所有者最近一次更新 Lease 的时间。
   // +optional
   RenewTime *metav1.MicroTime `json:"renewTime,omitempty" protobuf:"bytes,4,opt,name=renewTime"`
   // leaseTransitions是 Lease 之间的过渡次数
   // holders.
   // +optional
   LeaseTransitions *int32 `json:"leaseTransitions,omitempty" protobuf:"varint,5,opt,name=leaseTransitions"`
}
```

创建：

```http
POST /apis/coordination.k8s.io/v1/namespaces/{namespace}/leases
```

添加配置：

```http
PATCH /apis/coordination.k8s.io/v1/namespaces/{namespace}/leases/{name}
```

修改：

```http
PUT /apis/coordination.k8s.io/v1/namespaces/{namespace}/leases/{name}
```

删除：

```http
DELETE /apis/coordination.k8s.io/v1/namespaces/{namespace}/leases/{name}
```

删除一堆：

```http
DELETE /apis/coordination.k8s.io/v1/namespaces/{namespace}/leases
```

读取：

```http
GET /apis/coordination.k8s.io/v1/namespaces/{namespace}/leases/{name}
```

读取列表：

```http
GET /apis/coordination.k8s.io/v1/namespaces/{namespace}/leases
```

读取全部命名空间的列表：

```http
GET /apis/coordination.k8s.io/v1/leases
```











