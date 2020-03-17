# Kubernetes API 学习 （三）- Apps API

 可以通过以下命令来查看有哪些资源对象：

```bash
kubectl api-resources --api-group=apps
NAME                  SHORTNAMES   APIGROUP   NAMESPACED   KIND
controllerrevisions                apps       true         ControllerRevision
daemonsets            ds           apps       true         DaemonSet
deployments           deploy       apps       true         Deployment
replicasets           rs           apps       true         ReplicaSet
statefulsets          sts          apps       true         StatefulSet
```

源码位于 API 源码中的 apps 包。



## Deployment

对象结构：

```go
type Deployment struct {
   metav1.TypeMeta `json:",inline"`
   // Standard object metadata.
   // +optional
   metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

   // Specification of the desired behavior of the Deployment.
   // +optional
   Spec DeploymentSpec `json:"spec,omitempty" protobuf:"bytes,2,opt,name=spec"`

   // Most recently observed status of the Deployment.
   // +optional
   Status DeploymentStatus `json:"status,omitempty" protobuf:"bytes,3,opt,name=status"`
}
```

DeploymentSpec:

```go
type DeploymentSpec struct {
   // Pod 的数量
   Replicas *int32 `json:"replicas,omitempty" protobuf:"varint,1,opt,name=replicas"`

   // 选择 Pod 中的 Label ，必须选中
   Selector *metav1.LabelSelector `json:"selector" protobuf:"bytes,2,opt,name=selector"`

   // 定义 Pod 的地方
   Template v1.PodTemplateSpec `json:"template" protobuf:"bytes,3,opt,name=template"`

   // 使用新的pod替换现有pod的部署策略。包括 Recreate、RollingUpdate 
   Strategy DeploymentStrategy `json:"strategy,omitempty" patchStrategy:"retainKeys" protobuf:"bytes,4,opt,name=strategy"`

   // 新创建的 Pod 在没有任何容器崩溃的情况下准备就绪的最短秒数，以使其被视为可用。
   // 默认为0（pod一准备好就被认为可用）
   // +optional
   MinReadySeconds int32 `json:"minReadySeconds,omitempty" protobuf:"varint,5,opt,name=minReadySeconds"`

   // 允许回滚的 老的 ReplicaSets 的数量.
   // This is a pointer to distinguish between explicit zero and not specified.
   // 默认为10。
   // +optional
   RevisionHistoryLimit *int32 `json:"revisionHistoryLimit,omitempty" protobuf:"varint,6,opt,name=revisionHistoryLimit"`

   // 指示 Deployment 是否已暂停。
   // +optional
   Paused bool `json:"paused,omitempty" protobuf:"varint,7,opt,name=paused"`

   // 部署在被认为失败之前取得进展的最长时间（秒）。
   // 部署控制器将继续处理失败的部署，并且在部署状态中将出现ProgressDeadlineExceeded原因的情况。
   // 请注意，在暂停部署期间不会估计进度。默认为600秒。
   ProgressDeadlineSeconds *int32 `json:"progressDeadlineSeconds,omitempty" protobuf:"varint,9,opt,name=progressDeadlineSeconds"`
}
```

DeploymentStatus：

```go
type DeploymentStatus struct {
   // 部署控制器观察到的生成。
   // +optional
   ObservedGeneration int64 `json:"observedGeneration,omitempty" protobuf:"varint,1,opt,name=observedGeneration"`

   // 全部的 Pod 数量
   Replicas int32 `json:"replicas,omitempty" protobuf:"varint,2,opt,name=replicas"`

   // 没有处于 terminated 状态的 Pod 数量
   UpdatedReplicas int32 `json:"updatedReplicas,omitempty" protobuf:"varint,3,opt,name=updatedReplicas"`

   // Ready 的 Pod 的数量
   ReadyReplicas int32 `json:"readyReplicas,omitempty" protobuf:"varint,7,opt,name=readyReplicas"`

   // Ready 或者在 minReadySeconds 时间内的 Pod 数量
   AvailableReplicas int32 `json:"availableReplicas,omitempty" protobuf:"varint,4,opt,name=availableReplicas"`

   // 失败的 Pod 数量
   UnavailableReplicas int32 `json:"unavailableReplicas,omitempty" protobuf:"varint,5,opt,name=unavailableReplicas"`

   // 状态集合
   Conditions []DeploymentCondition `json:"conditions,omitempty" patchStrategy:"merge" patchMergeKey:"type" protobuf:"bytes,6,rep,name=conditions"`

   // Deployment 哈希冲突的计数. 当需要为最新的复制集创建名称时，部署控制器将此字段用作冲突避免机制。
   CollisionCount *int32 `json:"collisionCount,omitempty" protobuf:"varint,8,opt,name=collisionCount"`
}
```

创建：

```http
POST /apis/apps/v1/namespaces/{namespace}/deployments
```

增加配置：

```http
PATCH /apis/apps/v1/namespaces/{namespace}/deployments/{name}
```

修改配置：

```http
PUT /apis/apps/v1/namespaces/{namespace}/deployments/{name}
```

删除配置：

```http
DELETE /apis/apps/v1/namespaces/{namespace}/deployments/{name}
```

删除一堆：

```http
DELETE /apis/apps/v1/namespaces/{namespace}/deployments
```

读取：

```http
GET /apis/apps/v1/namespaces/{namespace}/deployments/{name}
```

读取列表：

```http
GET /apis/apps/v1/namespaces/{namespace}/deployments
```

读取 全部命名空间的：

```http
GET /apis/apps/v1/deployments
```

增加状态：

```http
PATCH /apis/apps/v1/namespaces/{namespace}/deployments/{name}/status
```

读取状态：

```http
GET /apis/apps/v1/namespaces/{namespace}/deployments/{name}/status
```

修改状态：

```http
PUT /apis/apps/v1/namespaces/{namespace}/deployments/{name}/status
```

读取扩展：

```http
GET /apis/apps/v1/namespaces/{namespace}/deployments/{name}/scale
```

修改扩展：

```http
PUT /apis/apps/v1/namespaces/{namespace}/deployments/{name}/scale
```

增加扩展：

```http
PATCH /apis/apps/v1/namespaces/{namespace}/deployments/{name}/scale
```



## ReplicaSet

对象结构：

```go
type ReplicaSet struct {
   metav1.TypeMeta `json:",inline"`

   // If the Labels of a ReplicaSet are empty, they are defaulted to
   // be the same as the Pod(s) that the ReplicaSet manages.
   // Standard object's metadata. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
   // +optional
   metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

   // Spec defines the specification of the desired behavior of the ReplicaSet.
   // More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
   // +optional
   Spec ReplicaSetSpec `json:"spec,omitempty" protobuf:"bytes,2,opt,name=spec"`

   // Status is the most recently observed status of the ReplicaSet.
   // This data may be out of date by some window of time.
   // Populated by the system.
   // Read-only.
   // More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
   // +optional
   Status ReplicaSetStatus `json:"status,omitempty" protobuf:"bytes,3,opt,name=status"`
}
```

ReplicaSetSpec:

```go
type ReplicaSetSpec struct {
   // Pod 数量，默认为1
   Replicas *int32 `json:"replicas,omitempty" protobuf:"varint,1,opt,name=replicas"`

   // 最小的 Ready 的 Pod 的数量
   MinReadySeconds int32 `json:"minReadySeconds,omitempty" protobuf:"varint,4,opt,name=minReadySeconds"`

   // 选中的 Pod 的 Label
   Selector *metav1.LabelSelector `json:"selector" protobuf:"bytes,2,opt,name=selector"`

   // Pod 模版
   Template v1.PodTemplateSpec `json:"template,omitempty" protobuf:"bytes,3,opt,name=template"`
}
```

ReplicaSetStatus：

```go
type ReplicaSetStatus struct {
   // Pod 的数量
   Replicas int32 `json:"replicas" protobuf:"varint,1,opt,name=replicas"`

   // 全部 Label 都匹配的 Pod 数量
   FullyLabeledReplicas int32 `json:"fullyLabeledReplicas,omitempty" protobuf:"varint,2,opt,name=fullyLabeledReplicas"`

   // 处于 Ready 状态的 Pod 的数量
   ReadyReplicas int32 `json:"readyReplicas,omitempty" protobuf:"varint,4,opt,name=readyReplicas"`

   // 可获得的复制份数
   AvailableReplicas int32 `json:"availableReplicas,omitempty" protobuf:"varint,5,opt,name=availableReplicas"`

   // ObservedGeneration反映了最近观察到的复制集的生成.
   ObservedGeneration int64 `json:"observedGeneration,omitempty" protobuf:"varint,3,opt,name=observedGeneration"`

   // 状态集合
   Conditions []ReplicaSetCondition `json:"conditions,omitempty" patchStrategy:"merge" patchMergeKey:"type" protobuf:"bytes,6,rep,name=conditions"`
}
```

创建：

```http
POST /apis/apps/v1/namespaces/{namespace}/replicasets
```

添加配置：

```http
PATCH /apis/apps/v1/namespaces/{namespace}/replicasets/{name}
```

修改：

```http
PUT /apis/apps/v1/namespaces/{namespace}/replicasets/{name}
```

删除：

```http
DELETE /apis/apps/v1/namespaces/{namespace}/replicasets/{name}
```

删除一堆：

```http
DELETE /apis/apps/v1/namespaces/{namespace}/replicasets
```

读取：

```http
GET /apis/apps/v1/namespaces/{namespace}/replicasets/{name}
```

读取列表：

```http
GET /apis/apps/v1/namespaces/{namespace}/replicasets
```

读取所有命名空间的列表：

```http
GET /apis/apps/v1/replicasets
```

添加状态：

```http
PATCH /apis/apps/v1/namespaces/{namespace}/replicasets/{name}/status
```

读取状态：

```http
GET /apis/apps/v1/namespaces/{namespace}/replicasets/{name}/status
```

修改状态：

```http
PUT /apis/apps/v1/namespaces/{namespace}/replicasets/{name}/status
```

读取扩展：

```http
GET /apis/apps/v1/namespaces/{namespace}/replicasets/{name}/scale
```

修改扩展：

```http
PUT /apis/apps/v1/namespaces/{namespace}/replicasets/{name}/scale
```

添加扩展：

```http
PATCH /apis/apps/v1/namespaces/{namespace}/replicasets/{name}/scale
```



## DaemonSet

*DaemonSet* 确保全部（或者某些）节点上运行一个 Pod 的副本。当有节点加入集群时， 也会为他们新增一个 Pod 。当有节点从集群移除时，这些 Pod 也会被回收。删除 DaemonSet 将会删除它创建的所有 Pod。

DaemonSet 的一些典型用法：

- 在每个节点上运行集群存储 DaemonSet，例如 `glusterd`、`ceph`。
- 在每个节点上运行日志收集 DaemonSet，例如 `fluentd`、`logstash`。
- 在每个节点上运行监控 DaemonSet，例如 [Prometheus Node Exporter](https://github.com/prometheus/node_exporter)、[Flowmill](https://github.com/Flowmill/flowmill-k8s/)、[Sysdig 代理](https://docs.sysdig.com/)、`collectd`、[Dynatrace OneAgent](https://www.dynatrace.com/technologies/kubernetes-monitoring/)、[AppDynamics 代理](https://docs.appdynamics.com/display/CLOUD/Container+Visibility+with+Kubernetes)、[Datadog 代理](https://docs.datadoghq.com/agent/kubernetes/daemonset_setup/)、[New Relic 代理](https://docs.newrelic.com/docs/integrations/kubernetes-integration/installation/kubernetes-installation-configuration)、Ganglia `gmond` 或者 [Instana 代理](https://www.instana.com/supported-integrations/kubernetes-monitoring/)。

一个简单的用法是在所有的节点上都启动一个 DaemonSet，将被作为每种类型的 daemon 使用。

一个稍微复杂的用法是单独对每种 daemon 类型使用多个 DaemonSet，但具有不同的标志， 并且对不同硬件类型具有不同的内存、CPU 要求。

对象结构：

```go
type DaemonSet struct {
   metav1.TypeMeta `json:",inline"`
   // Standard object's metadata.
   // More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
   // +optional
   metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

   // The desired behavior of this daemon set.
   // More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
   // +optional
   Spec DaemonSetSpec `json:"spec,omitempty" protobuf:"bytes,2,opt,name=spec"`

   // The current status of this daemon set. This data may be
   // out of date by some window of time.
   // Populated by the system.
   // Read-only.
   // More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
   // +optional
   Status DaemonSetStatus `json:"status,omitempty" protobuf:"bytes,3,opt,name=status"`
}
```

DaemonSetSpec:

```go
type DaemonSetSpec struct {
   // 选中的 Pod
   Selector *metav1.LabelSelector `json:"selector" protobuf:"bytes,1,opt,name=selector"`

   // 定义 Pod
   Template v1.PodTemplateSpec `json:"template" protobuf:"bytes,2,opt,name=template"`

   // 更新策略
   UpdateStrategy DaemonSetUpdateStrategy `json:"updateStrategy,omitempty" protobuf:"bytes,3,opt,name=updateStrategy"`

   // 最小的处于 Ready 状态的数量
   MinReadySeconds int32 `json:"minReadySeconds,omitempty" protobuf:"varint,4,opt,name=minReadySeconds"`

   // The number of old history to retain to allow rollback.
   // This is a pointer to distinguish between explicit zero and not specified.
   // Defaults to 10.
   // +optional
   RevisionHistoryLimit *int32 `json:"revisionHistoryLimit,omitempty" protobuf:"varint,6,opt,name=revisionHistoryLimit"`
}
```

DaemonSetStatus

```go
type DaemonSetStatus struct {
   // 当前被调度的数量
   CurrentNumberScheduled int32 `json:"currentNumberScheduled" protobuf:"varint,1,opt,name=currentNumberScheduled"`

   // 还没有被调度的数量
   NumberMisscheduled int32 `json:"numberMisscheduled" protobuf:"varint,2,opt,name=numberMisscheduled"`

   // 渴望被调度的数量
   DesiredNumberScheduled int32 `json:"desiredNumberScheduled" protobuf:"varint,3,opt,name=desiredNumberScheduled"`

   // Ready 的数量
   NumberReady int32 `json:"numberReady" protobuf:"varint,4,opt,name=numberReady"`

   // The most recent generation observed by the daemon set controller.
   // +optional
   ObservedGeneration int64 `json:"observedGeneration,omitempty" protobuf:"varint,5,opt,name=observedGeneration"`

   // 更新的次数
   UpdatedNumberScheduled int32 `json:"updatedNumberScheduled,omitempty" protobuf:"varint,6,opt,name=updatedNumberScheduled"`

   // 可获得的数量
   NumberAvailable int32 `json:"numberAvailable,omitempty" protobuf:"varint,7,opt,name=numberAvailable"`

   // 不能获得的数量
   NumberUnavailable int32 `json:"numberUnavailable,omitempty" protobuf:"varint,8,opt,name=numberUnavailable"`

   // Count of hash collisions for the DaemonSet. The DaemonSet controller
   // uses this field as a collision avoidance mechanism when it needs to
   // create the name for the newest ControllerRevision.
   // +optional
   CollisionCount *int32 `json:"collisionCount,omitempty" protobuf:"varint,9,opt,name=collisionCount"`

   // 状态列表
   Conditions []DaemonSetCondition `json:"conditions,omitempty" patchStrategy:"merge" patchMergeKey:"type" protobuf:"bytes,10,rep,name=conditions"`
}
```

创建：

```http
POST /apis/apps/v1/namespaces/{namespace}/daemonsets
```

添加配置：

```http
PATCH /apis/apps/v1/namespaces/{namespace}/daemonsets/{name}
```

修改配置：

```http
PUT /apis/apps/v1/namespaces/{namespace}/daemonsets/{name}
```

删除：

```http
DELETE /apis/apps/v1/namespaces/{namespace}/daemonsets/{name}
```

删除一堆：

```http
DELETE /apis/apps/v1/namespaces/{namespace}/daemonsets
```

读取：

```http
GET /apis/apps/v1/namespaces/{namespace}/daemonsets/{name}
```

读取列表：

```http
GET /apis/apps/v1/namespaces/{namespace}/daemonsets
```

读取全部命名空间的：

```http
GET /apis/apps/v1/daemonsets
```

添加状态：

```http
PATCH /apis/apps/v1/namespaces/{namespace}/daemonsets/{name}/status
```

读取状态：

```http
GET /apis/apps/v1/namespaces/{namespace}/daemonsets/{name}/status
```

修改状态：

```http
PUT /apis/apps/v1/namespaces/{namespace}/daemonsets/{name}/status
```



## StatefulSet

对象结构：

```go
type StatefulSet struct {
	metav1.TypeMeta `json:",inline"`
	// +optional
	metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

	// Spec defines the desired identities of pods in this set.
	// +optional
	Spec StatefulSetSpec `json:"spec,omitempty" protobuf:"bytes,2,opt,name=spec"`

	// Status is the current status of Pods in this StatefulSet. This data
	// may be out of date by some window of time.
	// +optional
	Status StatefulSetStatus `json:"status,omitempty" protobuf:"bytes,3,opt,name=status"`
}
```

StatefulSetSpec:

```go
type StatefulSetSpec struct {
   // Pod 数量
   Replicas *int32 `json:"replicas,omitempty" protobuf:"varint,1,opt,name=replicas"`

   // 标签选择器
   Selector *metav1.LabelSelector `json:"selector" protobuf:"bytes,2,opt,name=selector"`

   // Pod 模版
   Template v1.PodTemplateSpec `json:"template" protobuf:"bytes,3,opt,name=template"`

   // PVC列表
   VolumeClaimTemplates []v1.PersistentVolumeClaim `json:"volumeClaimTemplates,omitempty" protobuf:"bytes,4,rep,name=volumeClaimTemplates"`

   // 服务名称
   ServiceName string `json:"serviceName" protobuf:"bytes,5,opt,name=serviceName"`

   //podManagementPolicy控制在初始扩展期间如何创建pod，
   // 更换节点上的播客时，或缩小比例时。默认策略是
   // `OrderedReady`，其中pod是按递增顺序创建的（pod-0，然后pod-1，等等），控制器将等到每个pod都准备好后再继续。缩小比例时，按相反的顺序移除豆荚。
   // 另一种策略是“Parallel”，它将创建并行的pod以匹配所需的比例，而无需等待，并且在scale down时将立即删除所有pod。
   // podManagementPolicy controls how pods are created during initial scale up,
   // when replacing pods on nodes, or when scaling down. The default policy is
   // `OrderedReady`, where pods are created in increasing order (pod-0, then
   // pod-1, etc) and the controller will wait until each pod is ready before
   // continuing. When scaling down, the pods are removed in the opposite order.
   // The alternative policy is `Parallel` which will create pods in parallel
   // to match the desired scale without waiting, and on scale down will delete
   // all pods at once.
   // +optional
   PodManagementPolicy PodManagementPolicyType `json:"podManagementPolicy,omitempty" protobuf:"bytes,6,opt,name=podManagementPolicy,casttype=PodManagementPolicyType"`

   // 更新策略
   UpdateStrategy StatefulSetUpdateStrategy `json:"updateStrategy,omitempty" protobuf:"bytes,7,opt,name=updateStrategy"`

   // revisionHistoryLimit是将在StatefulSet的修订历史记录中维护的最大修订数。修订历史由当前应用的
   // StatefulSetSpec版本。默认值为10。
   RevisionHistoryLimit *int32 `json:"revisionHistoryLimit,omitempty" protobuf:"varint,8,opt,name=revisionHistoryLimit"`
}
```

StatefulSetStatus

```go
type StatefulSetStatus struct {
   // observedGeneration is the most recent generation observed for this StatefulSet. It corresponds to the
   // StatefulSet's generation, which is updated on mutation by the API Server.
   // +optional
   ObservedGeneration int64 `json:"observedGeneration,omitempty" protobuf:"varint,1,opt,name=observedGeneration"`

   // Pod 数量
   Replicas int32 `json:"replicas" protobuf:"varint,2,opt,name=replicas"`

   // Ready 状态的 Pod 数量
   ReadyReplicas int32 `json:"readyReplicas,omitempty" protobuf:"varint,3,opt,name=readyReplicas"`

   // 当前的 Pod 数量
   CurrentReplicas int32 `json:"currentReplicas,omitempty" protobuf:"varint,4,opt,name=currentReplicas"`

   // 更新过后的 Pod 数量
   UpdatedReplicas int32 `json:"updatedReplicas,omitempty" protobuf:"varint,5,opt,name=updatedReplicas"`

   // currentRevision, if not empty, indicates the version of the StatefulSet used to generate Pods in the
   // sequence [0,currentReplicas).
   CurrentRevision string `json:"currentRevision,omitempty" protobuf:"bytes,6,opt,name=currentRevision"`

   // updateRevision，如果不为空，则表示用于在序列中生成pod的StatefulSet的版本
   // [replicas-updatedReplicas,replicas)
   UpdateRevision string `json:"updateRevision,omitempty" protobuf:"bytes,7,opt,name=updateRevision"`

   // collisionCount is the count of hash collisions for the StatefulSet. The StatefulSet controller
   // uses this field as a collision avoidance mechanism when it needs to create the name for the
   // newest ControllerRevision.
   // +optional
   CollisionCount *int32 `json:"collisionCount,omitempty" protobuf:"varint,9,opt,name=collisionCount"`

   // 状态列表
   Conditions []StatefulSetCondition `json:"conditions,omitempty" patchStrategy:"merge" patchMergeKey:"type" protobuf:"bytes,10,rep,name=conditions"`
}
```

创建：

```http
POST /apis/apps/v1/namespaces/{namespace}/statefulsets
```

添加配置：

```http
PATCH /apis/apps/v1/namespaces/{namespace}/statefulsets/{name}
```

修改：

```http
PUT /apis/apps/v1/namespaces/{namespace}/statefulsets/{name}
```

删除：

```http
DELETE /apis/apps/v1/namespaces/{namespace}/statefulsets/{name}
```

删除一堆：

```http
DELETE /apis/apps/v1/namespaces/{namespace}/statefulsets
```

读取：

```http
GET /apis/apps/v1/namespaces/{namespace}/statefulsets/{name}
```

读取列表：

```http
GET /apis/apps/v1/namespaces/{namespace}/statefulsets
```

读取全部命名空间的：

```http
GET /apis/apps/v1/statefulsets
```

添加状态：

```http
PATCH /apis/apps/v1/namespaces/{namespace}/statefulsets/{name}/status
```

读取状态：

```http
GET /apis/apps/v1/namespaces/{namespace}/statefulsets/{name}/status
```

修改 状态：

```http
PUT /apis/apps/v1/namespaces/{namespace}/statefulsets/{name}/status
```

读取扩展：

```http
GET /apis/apps/v1/namespaces/{namespace}/statefulsets/{name}/scale
```

修改扩展：

```http
PUT /apis/apps/v1/namespaces/{namespace}/statefulsets/{name}/scale
```

添加扩展：

```http
PATCH /apis/apps/v1/namespaces/{namespace}/statefulsets/{name}/scale
```



## ControllerRevision

控制器版本修订数据

对象结构：

```go
type ControllerRevision struct {
   metav1.TypeMeta `json:",inline"`
   // Standard object's metadata.
   // More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
   // +optional
   metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

   // 数据是状态的序列化表示。
   Data runtime.RawExtension `json:"data,omitempty" protobuf:"bytes,2,opt,name=data"`

   // 表示由数据表示的状态的修订。
   Revision int64 `json:"revision" protobuf:"varint,3,opt,name=revision"`
}
```

创建：

```http
POST /apis/apps/v1/namespaces/{namespace}/controllerrevisions
```

添加配置：

```http
PATCH /apis/apps/v1/namespaces/{namespace}/controllerrevisions/{name}
```

修改：

```http
PUT /apis/apps/v1/namespaces/{namespace}/controllerrevisions/{name}
```

删除：

```http
DELETE /apis/apps/v1/namespaces/{namespace}/controllerrevisions/{name}
```

删除一堆：

```http
DELETE /apis/apps/v1/namespaces/{namespace}/controllerrevisions
```

读取：

```http
GET /apis/apps/v1/namespaces/{namespace}/controllerrevisions/{name}
```

读取列表 ：

```http
GET /apis/apps/v1/namespaces/{namespace}/controllerrevisions
```

读取全部命名空间的：

```http
GET /apis/apps/v1/controllerrevisions
```























