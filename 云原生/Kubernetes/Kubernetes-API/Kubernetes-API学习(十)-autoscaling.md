# Kubernetes API 学习(十) - autoscaling

查看资源对象列表：

```bash
$ kubectl api-resources --api-group=autoscaling
NAME                       SHORTNAMES   APIGROUP      NAMESPACED   KIND
horizontalpodautoscalers   hpa          autoscaling   true         HorizontalPodAutoscaler
```

只有一个 HorizontalPodAutoscaler

源码位于 API 源码包的 autoscaling 包中。

Pod 水平自动伸缩（Horizontal Pod Autoscaler）特性， 可以基于CPU利用率自动伸缩 replication controller、deployment和 replica set 中的 pod 数量，（除了 CPU 利用率）也可以 基于其他应程序提供的度量指标[custom metrics](https://git.k8s.io/community/contributors/design-proposals/instrumentation/custom-metrics-api.md)。 pod 自动缩放不适用于无法缩放的对象，比如 DaemonSets。

Pod 水平自动伸缩特性由 Kubernetes API 资源和控制器实现。资源决定了控制器的行为。 控制器会周期性的获取平均 CPU 利用率，并与目标值相比较后来调整 replication controller 或 deployment 中的副本数量。

对象结构：

```go
type HorizontalPodAutoscaler struct {
	metav1.TypeMeta `json:",inline"`
	// Standard object metadata. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
	// +optional
	metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

	// behaviour of autoscaler. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status.
	// +optional
	Spec HorizontalPodAutoscalerSpec `json:"spec,omitempty" protobuf:"bytes,2,opt,name=spec"`

	// current information about the autoscaler.
	// +optional
	Status HorizontalPodAutoscalerStatus `json:"status,omitempty" protobuf:"bytes,3,opt,name=status"`
}
```

HorizontalPodAutoscalerSpec:

```go
type HorizontalPodAutoscalerSpec struct {
   // 指定自动扩展的目标
   ScaleTargetRef CrossVersionObjectReference `json:"scaleTargetRef" protobuf:"bytes,1,opt,name=scaleTargetRef"`
   // 最小 Pod 数量
   MinReplicas *int32 `json:"minReplicas,omitempty" protobuf:"varint,2,opt,name=minReplicas"`
   // 最大 Pod 数量
   MaxReplicas int32 `json:"maxReplicas" protobuf:"varint,3,opt,name=maxReplicas"`
   // 所有pod的目标平均CPU利用率（表示为请求CPU的百分比）；
   //如果未指定，则将使用默认的自动缩放策略。
   TargetCPUUtilizationPercentage *int32 `json:"targetCPUUtilizationPercentage,omitempty" protobuf:"varint,4,opt,name=targetCPUUtilizationPercentage"`
}
```

HorizontalPodAutoscalerStatus

```go
type HorizontalPodAutoscalerStatus struct {
   // 这台自动定标器观测到的最新一代。
   // +optional
   ObservedGeneration *int64 `json:"observedGeneration,omitempty" protobuf:"varint,1,opt,name=observedGeneration"`

   // 最后 一次扩展时间
   LastScaleTime *metav1.Time `json:"lastScaleTime,omitempty" protobuf:"bytes,2,opt,name=lastScaleTime"`

   // 当前 Pod 数量
   CurrentReplicas int32 `json:"currentReplicas" protobuf:"varint,3,opt,name=currentReplicas"`

   // 期望的数量
   DesiredReplicas int32 `json:"desiredReplicas" protobuf:"varint,4,opt,name=desiredReplicas"`

   // 所有pod的当前平均CPU利用率，表示为请求CPU的百分比，
   // e.g. 70 means that an average pod is using now 70% of its requested CPU.
   // +optional
   CurrentCPUUtilizationPercentage *int32 `json:"currentCPUUtilizationPercentage,omitempty" protobuf:"varint,5,opt,name=currentCPUUtilizationPercentage"`
}
```

创建：

```http
POST /apis/autoscaling/v1/namespaces/{namespace}/horizontalpodautoscalers
```

添加配置：

```http
PATCH /apis/autoscaling/v1/namespaces/{namespace}/horizontalpodautoscalers/{name}
```

修改：

```http
PUT /apis/autoscaling/v1/namespaces/{namespace}/horizontalpodautoscalers/{name}
```

删除：

```http
DELETE /apis/autoscaling/v1/namespaces/{namespace}/horizontalpodautoscalers/{name}
```

删除一堆：

```http
DELETE /apis/autoscaling/v1/namespaces/{namespace}/horizontalpodautoscalers
```

读取：

```http
GET /apis/autoscaling/v1/namespaces/{namespace}/horizontalpodautoscalers/{name}
```

读取列表：

```http
GET /apis/autoscaling/v1/namespaces/{namespace}/horizontalpodautoscalers
```

读取全部命名空间的列表：

```http
GET /apis/autoscaling/v1/horizontalpodautoscalers
```

添加状态：

```http
PATCH /apis/autoscaling/v1/namespaces/{namespace}/horizontalpodautoscalers/{name}/status
```

读取状态：

```http
GET /apis/autoscaling/v1/namespaces/{namespace}/horizontalpodautoscalers/{name}/status
```

修改状态：

```http
PUT /apis/autoscaling/v1/namespaces/{namespace}/horizontalpodautoscalers/{name}/status
```

























