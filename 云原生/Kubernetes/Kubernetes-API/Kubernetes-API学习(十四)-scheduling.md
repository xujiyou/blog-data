# Kubernetes API 学习(十四) - scheduling

```bash
$ kubectl api-resources --api-group=scheduling.k8s.io
NAME              SHORTNAMES   APIGROUP            NAMESPACED   KIND
priorityclasses   pc           scheduling.k8s.io   false        PriorityClass
```

源码在 API 源码包的 scheduling 包中。



现在版本支持 Pod 优先级抢占，通过 PriorityClass 来实现同一个 Node 节点内部的 Pod 对象抢占。根据 Pod 中运行的作业类型判定各个 Pod 的优先级，对于高优先级的 Pod 可以抢占低优先级 Pod 的资源。Pod priority 指的是Pod的优先级，高优先级的Pod会优先被调度，或者在资源不足低情况牺牲低优先级的Pod，以便于重要的Pod能够得到资源部署.

当节点没有足够的资源供调度器调度Pod、导致Pod处于pending时，抢占（preemption）逻辑会被触发。Preemption会尝试从一个节点删除低优先级的Pod，从而释放资源使高优先级的Pod得到节点资源进行部署。

对象结构：

```go
type PriorityClass struct {
   metav1.TypeMeta `json:",inline"`
   // Standard object's metadata.
   // More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
   // +optional
   metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

   // 此优先级类别的值。 这是Pod在其Pod规范中具有此类的名称时获得的实际优先级。
   Value int32 `json:"value" protobuf:"bytes,2,opt,name=value"`

   // globalDefault指定对于没有任何优先级类别的Pod，是否应将此PriorityClass视为默认优先级。
   // 只能将一个PriorityClass标记为`globalDefault`。 
   //但是，如果存在多个“优先级类”，并且它们的“ globalDefault”字段设置为true，则将此类全局默认“优先级类”的最小值用作默认优先级。
   // +optional
   GlobalDefault bool `json:"globalDefault,omitempty" protobuf:"bytes,3,opt,name=globalDefault"`

  
   // description是一个任意字符串，通常为何时使用此优先级类提供指导。
   // +optional
   Description string `json:"description,omitempty" protobuf:"bytes,4,opt,name=description"`

   // PreemptionPolicy是抢占优先级较低的Pod的策略。
   // 从不，PreemptLowerPriority之一。
   // 如果未设置，则默认为PreemptLowerPriority。
   // 此字段是字母级别的，并且仅由启用NonPreemptingPriority功能的服务器使用。
   // PreemptionPolicy is the Policy for preempting pods with lower priority.
   // One of Never, PreemptLowerPriority.
   // Defaults to PreemptLowerPriority if unset.
   // This field is alpha-level and is only honored by servers that enable the NonPreemptingPriority feature.
   // +optional
   PreemptionPolicy *apiv1.PreemptionPolicy `json:"preemptionPolicy,omitempty" protobuf:"bytes,5,opt,name=preemptionPolicy"`
}
```

创建：

```
POST /apis/scheduling.k8s.io/v1/priorityclasses
```

添加配置：

```
PATCH /apis/scheduling.k8s.io/v1/priorityclasses/{name}
```

修改：

```
PUT /apis/scheduling.k8s.io/v1/priorityclasses/{name}
```

删除：

```
DELETE /apis/scheduling.k8s.io/v1/priorityclasses/{name}
```

删除一堆：

```
DELETE /apis/scheduling.k8s.io/v1/priorityclasses
```

读取：

```
GET /apis/scheduling.k8s.io/v1/priorityclasses/{name}
```

读取列表：

```
GET /apis/scheduling.k8s.io/v1/priorityclasses
```















