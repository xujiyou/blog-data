# Kubernetes API 学习（二）- 核心API

关于 Core API，它的 APIGROUP 为空，并且只有 `v1` 一个版本。

可以通过以下命令来查看有哪些资源对象：

```bash
$ kubectl api-resources --api-group=""
NAME                     SHORTNAMES   APIGROUP   NAMESPACED   KIND
bindings                                         true         Binding
componentstatuses        cs                      false        ComponentStatus
configmaps               cm                      true         ConfigMap
endpoints                ep                      true         Endpoints
events                   ev                      true         Event
limitranges              limits                  true         LimitRange
namespaces               ns                      false        Namespace
nodes                    no                      false        Node
persistentvolumeclaims   pvc                     true         PersistentVolumeClaim
persistentvolumes        pv                      false        PersistentVolume
pods                     po                      true         Pod
podtemplates                                     true         PodTemplate
replicationcontrollers   rc                      true         ReplicationController
resourcequotas           quota                   true         ResourceQuota
secrets                                          true         Secret
serviceaccounts          sa                      true         ServiceAccount
services                 svc                     true         Service
```

或者通过 curl 或 浏览器获取：

```http
GET /api/v1
```

共 17 个。

---



源码位于 API 源码中的 core 包。

Core API 的资源对象种类在 `core/v1/register.go` 中。

注意其中有很多 List 对象，但是使用 kubectl 并不能直接 get 这个 List 对象。这个 List 对象是啥那？

其实 List 对象是某种对象的返回格式，比如在执行 `kubelet get pod -o yaml` 时，返回的数据就是 PodList 对象。

各种对象的具体定义在 `core/v1/types.go` 中。

下面来看具体的对象类型。



## Pod

先来看一下 Pod 的对象格式。

```go
type Pod struct {
	metav1.TypeMeta `json:",inline"`
	metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`
	Spec PodSpec `json:"spec,omitempty" protobuf:"bytes,2,opt,name=spec"`
	Status PodStatus `json:"status,omitempty" protobuf:"bytes,3,opt,name=status"`
}
```

可以看到，Pod 一共有四种字段，简单明了，这里需要注意一下 json，和 protobuf 格式。

主要看 json 格式，`inline` 代表使用内部字段，比如这里的 `metav1.TypeMeta` 内部有：

```go
type TypeMeta struct {
	Kind string `json:"kind,omitempty" protobuf:"bytes,1,opt,name=kind"`
	APIVersion string `json:"apiVersion,omitempty" protobuf:"bytes,2,opt,name=apiVersion"`
}
```

`omitempty` 表示可以省略

### metav1.ObjectMeta

这里再看一下元数据对象，这里看了之后就不会再看了。

```go
type ObjectMeta struct {
	// Name 必须是 namespace 内唯一的. 在创建资源时他是必须的, 一些资源也会在创建时自动创建名称. 
	// 不能更新
	Name string `json:"name,omitempty" protobuf:"bytes,1,opt,name=name"`

	// GenerateName 是一个可选的前缀，被服务用于生成一个唯一的 name。
	// 只有在 Name 字段未指定时生效。
	// 如果使用此字段，则每次返回给客户端的名称都会不同
	// 这个 GenerateName 还会一个唯一的后缀组合
	// 最后生成的值和 Name 字段有相同的规则
	GenerateName string `json:"generateName,omitempty" protobuf:"bytes,2,opt,name=generateName"`

	// 指定命名空间，如不指定，则为 default，有些资源也不必指定这个字段。
	// 必须符合 DNS 的命名规则
	// 不可更新
	Namespace string `json:"namespace,omitempty" protobuf:"bytes,3,opt,name=namespace"`

	// SelfLink 是代表此对象的URL，由系统填充，只读
	// 已过期！！！
	SelfLink string `json:"selfLink,omitempty" protobuf:"bytes,4,opt,name=selfLink"`

	// 唯一ID，在资源创建成功后生成
	// Populated by the system.
	// 只读，由系统生成
	UID types.UID `json:"uid,omitempty" protobuf:"bytes,5,opt,name=uid,casttype=k8s.io/kubernetes/pkg/types.UID"`

	// 代表该对象内部版本，客户端可以使用该值来确定对象何时更改过。
  // 可用于乐观并发，更改检测以及对一个资源或一组资源的监视操作。
  // 客户端可以将其未修改地传递回服务器。 它们可能仅对特定资源或一组资源有效。
	// 只读，由系统生成
	ResourceVersion string `json:"resourceVersion,omitempty" protobuf:"bytes,6,opt,name=resourceVersion"`

	// 代表所需状态的特定生成的序列号。
	// 只读，由系统生成
	Generation int64 `json:"generation,omitempty" protobuf:"varint,7,opt,name=generation"`

	// 资源创建时间，它以RFC3339形式表示并且采用UTC。
	// 只读，由系统生成
	// 对 List 对象来说是 NULL
	CreationTimestamp Time `json:"creationTimestamp,omitempty" protobuf:"bytes,8,opt,name=creationTimestamp"`

	// 资源删除的时间，当用户请求正常删除时，此字段由服务器设置，客户端无法直接设置。
  // 一旦终结器列表为空，则预计在此字段中的时间之后，资源将被删除（从资源列表中不再可见，并且名称无法访问）。
  // 只要终结器列表中包含项目，删除就会被阻止。一旦设置了deleteTimestamp，
  // 尽管可能会缩短该值或在此时间之前删除资源，但可能不会取消设置该值，也不会在以后设置它。
  // 只读，由系统生成
	DeletionTimestamp *Time `json:"deletionTimestamp,omitempty" protobuf:"bytes,9,opt,name=deletionTimestamp"`

	// 在从系统中删除该对象之前，该对象正常终止所允许的秒数。 仅在设置了deleteTimestamp时设置。
	// 只读
	DeletionGracePeriodSeconds *int64 `json:"deletionGracePeriodSeconds,omitempty" protobuf:"varint,10,opt,name=deletionGracePeriodSeconds"`

	// 字符串键和值的 Map，可用于组织和分类（范围和选择）对象。 可以匹配控制器和 Service 的选择器。
	Labels map[string]string `json:"labels,omitempty" protobuf:"bytes,11,rep,name=labels"`

	//注释是与资源一起存储的非结构化键值映射，可以由外部工具设置该资源来存储和检索任意元数据。 
  // 它们不可查询，在修改对象时应保留它们。
	Annotations map[string]string `json:"annotations,omitempty" protobuf:"bytes,12,rep,name=annotations"`

	// 该对象所依赖的对象列表。 如果列表中的所有对象都被删除后，该对象将被垃圾回收。 
  // 如果此对象由控制器管理，则此列表中的条目将指向该控制器，并且controller字段设置为true。
  // 最多只能有一个管理控制器。
	// +optional
	// +patchMergeKey=uid
	// +patchStrategy=merge
	OwnerReferences []OwnerReference `json:"ownerReferences,omitempty" patchStrategy:"merge" patchMergeKey:"uid" protobuf:"bytes,13,rep,name=ownerReferences"`

	// 从注册表中删除对象之前，必须为空。 每个条目都是负责组件的标识符，该组件将从列表中删除该条目。 
  // 如果对象的deleteTimestamp不为nil，则只能删除此列表中的条目。 
  // 终结器可以按任何顺序处理和删除。 不强制执行订单，因为这会带来终结器卡住的巨大风险。 
  // finalizers是一个共享字段，任何获得许可的演员都可以对其重新排序。 
  // 如果终结器列表是按顺序处理的，则可能导致以下情况：列表中负责第一个终结器的组件正在等待由负责终结器的组件产生的信号（字段值，外部系统或其他）。 
  // 终结器在列表的后面，导致死锁。 如果没有强制订购，终结者可以自由地在自己之间订购，并且不易受到列表中订购变更的影响。
	Finalizers []string `json:"finalizers,omitempty" patchStrategy:"merge" protobuf:"bytes,14,rep,name=finalizers"`

	// 对象所属的集群的名称。 这用于区分不同集群中具有相同名称和名称空间的资源。 
  // 目前尚未在任何地方设置此字段，并且如果在create或update请求中设置，则apiserver将忽略此字段。
	ClusterName string `json:"clusterName,omitempty" protobuf:"bytes,15,opt,name=clusterName"`

	// ManagedFields将工作流ID和版本映射到该工作流管理的字段集。 
  // 这主要用于内部整理，用户通常不需要设置或了解此字段。 
  // 工作流可以是用户名，控制器名或特定应用路径的名称，例如“ ci-cd”。 字段集始终是修改对象时工作流使用的版本。
	ManagedFields []ManagedFieldsEntry `json:"managedFields,omitempty" protobuf:"bytes,17,rep,name=managedFields"`
}
```

下面看一下在 Metadata 中提到的 OwnerReference：

```go
type OwnerReference struct {
	APIVersion string `json:"apiVersion" protobuf:"bytes,5,opt,name=apiVersion"`
	Kind string `json:"kind" protobuf:"bytes,1,opt,name=kind"`
	Name string `json:"name" protobuf:"bytes,3,opt,name=name"`
	UID types.UID `json:"uid" protobuf:"bytes,4,opt,name=uid,casttype=k8s.io/apimachinery/pkg/types.UID"`
	Controller *bool `json:"controller,omitempty" protobuf:"varint,6,opt,name=controller"`
	BlockOwnerDeletion *bool `json:"blockOwnerDeletion,omitempty" protobuf:"varint,7,opt,name=blockOwnerDeletion"`
}
```

比如在创建 Deployment 时，自动创建了的 Pod 里边就有这个字段。

看完了元数据，下面重点看一下 PodSpec，一共 34 个字段 ：

```go
type PodSpec struct {
	// Pod 可以挂载的卷列表
	Volumes []Volume `json:"volumes,omitempty" patchStrategy:"merge,retainKeys" patchMergeKey:"name" protobuf:"bytes,1,rep,name=volumes"`
	// Pod 的初始化容器列表
	// 初始化容器在容器启动之前按顺序执行. 如果有容器失败了，将按照重启策略重启
  // 初始化容器或普通容器的名称在所有容器中必须唯一。
	// 初始化容器没有Lifecycle操作，Readiness探针，Liveness探针或Startup探针。
	// 在调度期间，通过找到每种资源类型的最高请求/限制，然后使用该值的最大值或普通容器的总和，
  // 来考虑初始化容器的resourceRequirements。 限制以类似的方式应用于初始化容器。
	// 无法单独添加或删除初始化容器。
	// 不能更新
	InitContainers []Container `json:"initContainers,omitempty" patchStrategy:"merge" patchMergeKey:"name" protobuf:"bytes,20,rep,name=initContainers"`
	// 属于该 Pod 的容器列表。
	// 无法单独添加或删除容器。
	// Pod 中至少有一个容器
	// 不能更新
	Containers []Container `json:"containers" patchStrategy:"merge" patchMergeKey:"name" protobuf:"bytes,2,rep,name=containers"`
	// 在此容器中运行的临时容器列表。 临时容器可以在现有的容器中运行，以执行用户启动的操作，例如调试。 
  // 创建Pod时无法指定此列表，也无法通过更新Pod规范进行修改。 
  // 为了向现有容器添加临时容器，请使用容器的ephemeralcontainers子资源。 
  // 此字段为Alpha级别，只有启用EphemeralContainers功能的服务器才能使用。
	EphemeralContainers []EphemeralContainer `json:"ephemeralContainers,omitempty" patchStrategy:"merge" patchMergeKey:"name" protobuf:"bytes,34,rep,name=ephemeralContainers"`
	// 重启策略。可以是：Always, OnFailure, Never.默认是 Always
	RestartPolicy RestartPolicy `json:"restartPolicy,omitempty" protobuf:"bytes,3,opt,name=restartPolicy,casttype=RestartPolicy"`
	// Pod 需要正常终止的可选持续时间（以秒为单位）。 在删除请求中可能会减少。 必须为非负整数。 零值表示立即删除。
  // 如果此值为nil，则将使用默认宽限期。
  // 宽限期是指在吊舱中运行的进程被发送终止信号后的持续时间（以秒为单位），以及进程被终止信号强制终止的时间。
  // 将此值设置为比您的进程的预期清除时间长。
  // 默认为30秒。
	TerminationGracePeriodSeconds *int64 `json:"terminationGracePeriodSeconds,omitempty" protobuf:"varint,4,opt,name=terminationGracePeriodSeconds"`
	// 相对于节点，节点上的pod可能处于活动状态的可选持续时间（以秒为单位）
  // 系统将主动尝试将其标记为失败并终止关联容器之前的StartTime。
  // 此值必须为正整数。
	ActiveDeadlineSeconds *int64 `json:"activeDeadlineSeconds,omitempty" protobuf:"varint,5,opt,name=activeDeadlineSeconds"`
	// 设置 Pod 中的 DNS 策略，默认是 ClusterFirst
	// 可选值是 'ClusterFirstWithHostNet', 'ClusterFirst', 'Default' or 'None'.
	// DNSConfig中提供的DNS参数将与通过DNSPolicy选择的策略合并。
  // 要与hostNetwork一起设置DNS选项，必须将DNS策略明确指定为'ClusterFirstWithHostNet'。
	DNSPolicy DNSPolicy `json:"dnsPolicy,omitempty" protobuf:"bytes,6,opt,name=dnsPolicy,casttype=DNSPolicy"`
	// 为 Pod 选择 node，这里填 node 的 label列表。
	NodeSelector map[string]string `json:"nodeSelector,omitempty" protobuf:"bytes,7,rep,name=nodeSelector"`
	// ServiceAccount 名，用于提供访问 kube-apiserver 的访问权限
	ServiceAccountName string `json:"serviceAccountName,omitempty" protobuf:"bytes,8,opt,name=serviceAccountName"`
	// 过期作废了
	DeprecatedServiceAccount string `json:"serviceAccount,omitempty" protobuf:"bytes,9,opt,name=serviceAccount"`
	// 是否应自动安装服务帐户 token。
	AutomountServiceAccountToken *bool `json:"automountServiceAccountToken,omitempty" protobuf:"varint,21,opt,name=automountServiceAccountToken"`
	// NodeName是将此pod调度到特定节点上的请求。 如果是非空的
  // 假设它满足资源需求，则调度程序仅将此Pod调度到该节点上。
  // NodeSelector 的精简版
	NodeName string `json:"nodeName,omitempty" protobuf:"bytes,10,opt,name=nodeName"`
	// 使用 Pod 所在主机的网络
  // 如果设置了此选项，则必须指定将使用的端口。
	// 默认是 false.
	HostNetwork bool `json:"hostNetwork,omitempty" protobuf:"varint,11,opt,name=hostNetwork"`
	// 使用 Pod 所在主机的 pid namespace.这样在主机上和 Pod 内部看到的 pid 将是一样的。
	// 默认是 false.
	HostPID bool `json:"hostPID,omitempty" protobuf:"varint,12,opt,name=hostPID"`
	// 使用 Pod 所在主机的 ipc namespace.
	// 默认是 false.
	HostIPC bool `json:"hostIPC,omitempty" protobuf:"varint,13,opt,name=hostIPC"`
	// 在容器中的所有容器之间共享一个进程名称空间。
  // 设置此选项后，容器将能够查看其他容器并向其发出信号
  // 在同一容器中，每个容器中的第一个进程将不会分配PID 1。
  // 不能同时设置HostPID和ShareProcessNamespace。
	// 默认是 false.
	ShareProcessNamespace *bool `json:"shareProcessNamespace,omitempty" protobuf:"varint,27,opt,name=shareProcessNamespace"`
	// SecurityContext拥有容器级别的安全属性和通用容器设置。
	// 默认为空
	// +optional
	SecurityContext *PodSecurityContext `json:"securityContext,omitempty" protobuf:"bytes,14,opt,name=securityContext"`
	// ImagePullSecrets是在同一名称空间中用于秘密的引用的可选列表，可用于提取此PodSpec使用的任何镜像。
	// More info: https://kubernetes.io/docs/concepts/containers/images#specifying-imagepullsecrets-on-a-pod
	ImagePullSecrets []LocalObjectReference `json:"imagePullSecrets,omitempty" patchStrategy:"merge" patchMergeKey:"name" protobuf:"bytes,15,rep,name=imagePullSecrets"`
	// 指定 Pod 中的 hostname
	// 如果未指定，则 Pod 的主机名将设置为系统定义的值。
	// +optional
	Hostname string `json:"hostname,omitempty" protobuf:"bytes,16,opt,name=hostname"`
	// 如果指定，则标准Pod主机名将为 "<hostname>.<subdomain>.<pod namespace>.svc.<cluster domain>".
	// 如果未指定，则 Pod 没有域名。
	// +optional
	Subdomain string `json:"subdomain,omitempty" protobuf:"bytes,17,opt,name=subdomain"`
	// Pod 的调度约束，Affinity：亲和力
	Affinity *Affinity `json:"affinity,omitempty" protobuf:"bytes,18,opt,name=affinity"`
	// 如果指定，则将由指定的调度程序调度pod。
	// 如果未指定，则默认调度程序将调度pod。
	SchedulerName string `json:"schedulerName,omitempty" protobuf:"bytes,19,opt,name=schedulerName"`
	// node 的排斥性
	Tolerations []Toleration `json:"tolerations,omitempty" protobuf:"bytes,22,opt,name=tolerations"`
	// HostAliases是主机和IP的可选列表，将注入到Pod的主机中
	// 仅仅对 non-hostNetwork pods 有效
	HostAliases []HostAlias `json:"hostAliases,omitempty" patchStrategy:"merge" patchMergeKey:"ip" protobuf:"bytes,23,rep,name=hostAliases"`
	// Pod 的优先级，见 scheduling API
  // “system-node-critical”和“system-cluster-critical”是两个特殊的关键字，它们指示最高优先级. 
	PriorityClassName string `json:"priorityClassName,omitempty" protobuf:"bytes,24,opt,name=priorityClassName"`
	// 优先级值。 各种系统组件都使用此字段来查找窗格的优先级。 
  // 启用优先录入控制器后，它将阻止用户设置此字段。 
  // 准入控制器从PriorityClassName填充此字段。 值越高，优先级越高。
	Priority *int32 `json:"priority,omitempty" protobuf:"bytes,25,opt,name=priority"`
	// 指定容器的DNS参数。
  // 此处指定的参数将合并到生成的DNS中
  // 基于DNSPolicy的配置。
	DNSConfig *PodDNSConfig `json:"dnsConfig,omitempty" protobuf:"bytes,26,opt,name=dnsConfig"`
	// 如果指定，将评估所有 Pod 是否具有就绪状态。
	// More info: https://git.k8s.io/enhancements/keps/sig-network/0007-pod-ready%2B%2B.md
	ReadinessGates []PodReadinessGate `json:"readinessGates,omitempty" protobuf:"bytes,28,opt,name=readinessGates"`
	// RuntimeClassName引用node.k8s.io组中的RuntimeClass对象，该对象应用于运行此pod。 如果没有RuntimeClass资源与命名类匹配，则pod将不会运行。
  // 如果未设置或为空，则将使用“legacy” RuntimeClass，这是具有空定义的隐式类，使用默认的运行时处理程序。
	RuntimeClassName *string `json:"runtimeClassName,omitempty" protobuf:"bytes,29,opt,name=runtimeClassName"`
	// EnableServiceLinks指示是否应将与服务相关的信息注入到pod的环境变量中，以匹配Docker链接的语法。
	// Optional: Defaults to true.
	EnableServiceLinks *bool `json:"enableServiceLinks,omitempty" protobuf:"varint,30,opt,name=enableServiceLinks"`
	// PreemptionPolicy是抢占优先级较低的Pod的策略。
  // 如果未设置，则默认为PreemptLowerPriority。
  // 此字段是字母级别的，并且仅由启用NonPreemptingPriority功能的服务器使用。
	PreemptionPolicy *PreemptionPolicy `json:"preemptionPolicy,omitempty" protobuf:"bytes,31,opt,name=preemptionPolicy"`
	// Overhead represents the resource overhead associated with running a pod for a given RuntimeClass.
	// This field will be autopopulated at admission time by the RuntimeClass admission controller. If
	// the RuntimeClass admission controller is enabled, overhead must not be set in Pod create requests.
	// The RuntimeClass admission controller will reject Pod create requests which have the overhead already
	// set. If RuntimeClass is configured and selected in the PodSpec, Overhead will be set to the value
	// defined in the corresponding RuntimeClass, otherwise it will remain unset and treated as zero.
	// More info: https://git.k8s.io/enhancements/keps/sig-node/20190226-pod-overhead.md
	// This field is alpha-level as of Kubernetes v1.16, and is only honored by servers that enable the PodOverhead feature.
	// +optional
	Overhead ResourceList `json:"overhead,omitempty" protobuf:"bytes,32,opt,name=overhead"`
	// TopologySpreadConstraints describes how a group of pods ought to spread across topology
	// domains. Scheduler will schedule pods in a way which abides by the constraints.
	// This field is alpha-level and is only honored by clusters that enables the EvenPodsSpread
	// feature.
	// All topologySpreadConstraints are ANDed.
	// +optional
	// +patchMergeKey=topologyKey
	// +patchStrategy=merge
	// +listType=map
	// +listMapKey=topologyKey
	// +listMapKey=whenUnsatisfiable
	TopologySpreadConstraints []TopologySpreadConstraint `json:"topologySpreadConstraints,omitempty" patchStrategy:"merge" patchMergeKey:"topologyKey" protobuf:"bytes,33,opt,name=topologySpreadConstraints"`
}
```



下面就是 PodStatus 了。

```go
type PodStatus struct {
	// 用于显示 Pod 的状态，有 "Pending"，"Running"，"Succeeded"， "Failed"，"Unknown"
	// More info: https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle#pod-phase
	// +optional
	Phase PodPhase `json:"phase,omitempty" protobuf:"bytes,1,opt,name=phase,casttype=PodPhase"`
	// Pod 的状态列表
	Conditions []PodCondition `json:"conditions,omitempty" patchStrategy:"merge" patchMergeKey:"type" protobuf:"bytes,2,rep,name=conditions"`
	// 人类可读的 Pod 的细节信息
	// +optional
	Message string `json:"message,omitempty" protobuf:"bytes,3,opt,name=message"`
	// 一个简单的信息，标明为什么 Pod 会处于这个状态
	// e.g. 'Evicted'
	// +optional
	Reason string `json:"reason,omitempty" protobuf:"bytes,4,opt,name=reason"`
	// nominatedNodeName is set only when this pod preempts other pods on the node, but it cannot be
	// scheduled right away as preemption victims receive their graceful termination periods.
	// This field does not guarantee that the pod will be scheduled on this node. Scheduler may decide
	// to place the pod elsewhere if other nodes become available sooner. Scheduler may also decide to
	// give the resources on this node to a higher priority pod that is created after preemption.
	// As a result, this field may be different than PodSpec.nodeName when the pod is
	// scheduled.
	// +optional
	NominatedNodeName string `json:"nominatedNodeName,omitempty" protobuf:"bytes,11,opt,name=nominatedNodeName"`

	// Pod 所在主机的IP
	HostIP string `json:"hostIP,omitempty" protobuf:"bytes,5,opt,name=hostIP"`
	// Pod 自己的 IP
	PodIP string `json:"podIP,omitempty" protobuf:"bytes,6,opt,name=podIP"`

	// Pod 的 IP 列表，包含 上边的 PodIP
	PodIPs []PodIP `json:"podIPs,omitempty" protobuf:"bytes,12,rep,name=podIPs" patchStrategy:"merge" patchMergeKey:"ip"`

	// 创建时间
	StartTime *metav1.Time `json:"startTime,omitempty" protobuf:"bytes,7,opt,name=startTime"`

	// 初始化容器的信息
	InitContainerStatuses []ContainerStatus `json:"initContainerStatuses,omitempty" protobuf:"bytes,10,rep,name=initContainerStatuses"`

	// Pod 中容器信息
	ContainerStatuses []ContainerStatus `json:"containerStatuses,omitempty" protobuf:"bytes,8,rep,name=containerStatuses"`
	// The Quality of Service (QOS) classification assigned to the pod based on resource requirements
	// See PodQOSClass type for available QOS classes
	// More info: https://git.k8s.io/community/contributors/design-proposals/node/resource-qos.md
  // Guaranteed（放心），Burstable（稳定的），BestEffort（尽最大努力）
	// +optional
	QOSClass PodQOSClass `json:"qosClass,omitempty" protobuf:"bytes,9,rep,name=qosClass"`
	// Status for any ephemeral containers that have run in this pod.
	// This field is alpha-level and is only populated by servers that enable the EphemeralContainers feature.
	// +optional
	EphemeralContainerStatuses []ContainerStatus `json:"ephemeralContainerStatuses,omitempty" protobuf:"bytes,13,rep,name=ephemeralContainerStatuses"`
}
```

---

另外，Pod有哪些动作那 ，使用 `kubectl api-resources --api-group="" -o wide` 就可以看到了，Pod的动作有：

```
[create delete deletecollection get list patch update watch]
```

创建 POD，数据部分是 body：Pod：

```http
POST /api/v1/namespaces/{namespace}/pods
```

PATCH POD，数据部分为 body: Patch：

```http
PATCH /api/v1/namespaces/{namespace}/pods/{name}
```

Replace Pod，数据部分为 body: Pod：

```http
PUT /api/v1/namespaces/{namespace}/pods/{name}
```

Delete Pod：

```http
DELETE /api/v1/namespaces/{namespace}/pods/{name}
```

Delete Collection Pod:

```http
DELETE /api/v1/namespaces/{namespace}/pods
```

---

READ Pod:

```http
GET /api/v1/namespaces/{namespace}/pods/{name}
```

LIST Pod:

```http
GET /api/v1/namespaces/{namespace}/pods
```

List ALL Namespace Pod:

```http
GET /api/v1/pods
```

WATCH Pod:

```http
GET /api/v1/watch/namespaces/{namespace}/pods/{name}
```

WATCH List Pod:

```http
GET /api/v1/watch/namespaces/{namespace}/pods
```

Watch All Namespace Pod:

```http
GET /api/v1/watch/pods
```

更新 Pod 的状态：

```http
PATCH /api/v1/namespaces/{namespace}/pods/{name}/status
```

读取 Pod 的状态

```http
GET /api/v1/namespaces/{namespace}/pods/{name}/status
```

替换 Pod 的状态：

```
PUT /api/v1/namespaces/{namespace}/pods/{name}/status
```

---

查看 Pod 日志：

```http
GET /api/v1/namespaces/{namespace}/pods/{name}/log
```



## Service

首先看下 Service 对象的结构：

```go
type Service struct {
	metav1.TypeMeta `json:",inline"`
	// Standard object's metadata.
	// More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
	// +optional
	metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

	// Spec defines the behavior of a service.
	// https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
	// +optional
	Spec ServiceSpec `json:"spec,omitempty" protobuf:"bytes,2,opt,name=spec"`

	// Most recently observed status of the service.
	// Populated by the system.
	// Read-only.
	// More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
	// +optional
	Status ServiceStatus `json:"status,omitempty" protobuf:"bytes,3,opt,name=status"`
}
```

TypeMeta 和 ObjectMeta 不看了，介绍 Pod 时都说了。主要看 ServiceSpec：

```go
type ServiceSpec struct {
   // 端口列表，很熟悉了
   Ports []ServicePort `json:"ports,omitempty" patchStrategy:"merge" patchMergeKey:"port" protobuf:"bytes,1,rep,name=ports"`

   // 选择了哪些 Pod
   Selector map[string]string `json:"selector,omitempty" protobuf:"bytes,2,rep,name=selector"`

   // service 用到的 IP 地址，这个地址会写入到 iptables 和 ipvs
   ClusterIP string `json:"clusterIP,omitempty" protobuf:"bytes,3,opt,name=clusterIP"`

   // 也很熟悉了，ClusterIP、NodePort、LoadBalancer、ExternalName
   Type ServiceType `json:"type,omitempty" protobuf:"bytes,4,opt,name=type,casttype=ServiceType"`

   // 外部 IP，详情请看 https://kubernetes.io/zh/docs/concepts/services-networking/service/ 
   // 外部 IP 部分
   ExternalIPs []string `json:"externalIPs,omitempty" protobuf:"bytes,5,rep,name=externalIPs"`

   // Supports "ClientIP" and "None". Used to maintain session affinity.
   // Enable client IP based session affinity.
   // Must be ClientIP or None.
   // Defaults to None.
   // More info: https://kubernetes.io/docs/concepts/services-networking/service/#virtual-ips-and-service-proxies
   // +optional
   SessionAffinity ServiceAffinity `json:"sessionAffinity,omitempty" protobuf:"bytes,7,opt,name=sessionAffinity,casttype=ServiceAffinity"`

   // 负载均衡 IP 地址
   LoadBalancerIP string `json:"loadBalancerIP,omitempty" protobuf:"bytes,8,opt,name=loadBalancerIP"`

   // 负载均衡 IP 地址范围
   LoadBalancerSourceRanges []string `json:"loadBalancerSourceRanges,omitempty" protobuf:"bytes,9,opt,name=loadBalancerSourceRanges"`

   // 用于 dns 解析的名字
   ExternalName string `json:"externalName,omitempty" protobuf:"bytes,10,opt,name=externalName"`

   // externalTrafficPolicy denotes if this Service desires to route external
   // traffic to node-local or cluster-wide endpoints. "Local" preserves the
   // client source IP and avoids a second hop for LoadBalancer and Nodeport
   // type services, but risks potentially imbalanced traffic spreading.
   // "Cluster" obscures the client source IP and may cause a second hop to
   // another node, but should have good overall load-spreading.
   // +optional
   ExternalTrafficPolicy ServiceExternalTrafficPolicyType `json:"externalTrafficPolicy,omitempty" protobuf:"bytes,11,opt,name=externalTrafficPolicy"`

   // 用于健康检查的端口
   HealthCheckNodePort int32 `json:"healthCheckNodePort,omitempty" protobuf:"bytes,12,opt,name=healthCheckNodePort"`

   // 这个要注意，用于 Headless Service，表示将 service 下面的 Endpoints 也用上 DNS。
   PublishNotReadyAddresses bool `json:"publishNotReadyAddresses,omitempty" protobuf:"varint,13,opt,name=publishNotReadyAddresses"`

   // sessionAffinityConfig contains the configurations of session affinity.
   // +optional
   SessionAffinityConfig *SessionAffinityConfig `json:"sessionAffinityConfig,omitempty" protobuf:"bytes,14,opt,name=sessionAffinityConfig"`

   // ipv4 还是 ipv6
   IPFamily *IPFamily `json:"ipFamily,omitempty" protobuf:"bytes,15,opt,name=ipFamily,Configcasttype=IPFamily"`

   // topologyKeys is a preference-order list of topology keys which
   // implementations of services should use to preferentially sort endpoints
   // when accessing this Service, it can not be used at the same time as
   // externalTrafficPolicy=Local.
   // Topology keys must be valid label keys and at most 16 keys may be specified.
   // Endpoints are chosen based on the first topology key with available backends.
   // If this field is specified and all entries have no backends that match
   // the topology of the client, the service has no backends for that client
   // and connections should fail.
   // The special value "*" may be used to mean "any topology". This catch-all
   // value, if used, only makes sense as the last value in the list.
   // If this is not specified or empty, no topology constraints will be applied.
   // +optional
   TopologyKeys []string `json:"topologyKeys,omitempty" protobuf:"bytes,16,opt,name=topologyKeys"`
}
```

然后就是 ServiceStatus，这个很简单：

```go
type ServiceStatus struct {
	// LoadBalancer contains the current status of the load-balancer,
	// if one is present.
	// +optional
	LoadBalancer LoadBalancerStatus `json:"loadBalancer,omitempty" protobuf:"bytes,1,opt,name=loadBalancer"`
}
```



---



Service 的动作：

创建 Service：

```http
POST /api/v1/namespaces/{namespace}/services
```

更新 Service：

```http
PATCH /api/v1/namespaces/{namespace}/services/{name}
```

替换 Service：

```http
PUT /api/v1/namespaces/{namespace}/services/{name}
```

删除 Service：

```http
DELETE /api/v1/namespaces/{namespace}/services/{name}
```

读取 Service：

```http
GET /api/v1/namespaces/{namespace}/services/{name}
```

读取 Service 列表：

```http
GET /api/v1/namespaces/{namespace}/services
```

读取所有命名空间的 Service 列表：

```http
GET /api/v1/services
```

Watch Service:

```http
GET /api/v1/watch/namespaces/{namespace}/services/{name}
```

Watch Service 列表：

```http
GET /api/v1/watch/namespaces/{namespace}/services
```

Watch 全部命名空间的 Service 列表：

```http
GET /api/v1/watch/services
```

修改 Service 的状态：

```http
PATCH /api/v1/namespaces/{namespace}/services/{name}/status
```

读取 Service 的状态：

```http
GET /api/v1/namespaces/{namespace}/services/{name}/status
```

替换 Service 的状态：

```http
PUT /api/v1/namespaces/{namespace}/services/{name}/status
```



## Endpoints

先看资源对象结构：

```go
type Endpoints struct {
	metav1.TypeMeta `json:",inline"`
	// Standard object's metadata.
	// More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
	// +optional
	metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

	// The set of all endpoints is the union of all subsets. Addresses are placed into
	// subsets according to the IPs they share. A single address with multiple ports,
	// some of which are ready and some of which are not (because they come from
	// different containers) will result in the address being displayed in different
	// subsets for the different ports. No address will appear in both Addresses and
	// NotReadyAddresses in the same subset.
	// Sets of addresses and ports that comprise a service.
	// +optional
	Subsets []EndpointSubset `json:"subsets,omitempty" protobuf:"bytes,2,rep,name=subsets"`
}
```

看 EndpointSubset：

```go
type EndpointSubset struct {
	// IP addresses which offer the related ports that are marked as ready. These endpoints
	// should be considered safe for load balancers and clients to utilize.
	// +optional
	Addresses []EndpointAddress `json:"addresses,omitempty" protobuf:"bytes,1,rep,name=addresses"`
	// IP addresses which offer the related ports but are not currently marked as ready
	// because they have not yet finished starting, have recently failed a readiness check,
	// or have recently failed a liveness check.
	// +optional
	NotReadyAddresses []EndpointAddress `json:"notReadyAddresses,omitempty" protobuf:"bytes,2,rep,name=notReadyAddresses"`
	// Port numbers available on the related IP addresses.
	// +optional
	Ports []EndpointPort `json:"ports,omitempty" protobuf:"bytes,3,rep,name=ports"`
}
```

很好理解，不多解释了，下面看 API：

创建 Endpoints：

```http
POST /api/v1/namespaces/{namespace}/endpoints
```

修改 Endpoints：

```http
PATCH /api/v1/namespaces/{namespace}/endpoints/{name}
```

替换 Endpoints：

```http
PUT /api/v1/namespaces/{namespace}/endpoints/{name}
```

删除 Endpoints：

```http
DELETE /api/v1/namespaces/{namespace}/endpoints/{name}
```

集体删除：

```http
DELETE /api/v1/namespaces/{namespace}/endpoints
```

读取 Endpoints：

```http
GET /api/v1/namespaces/{namespace}/endpoints/{name}
```

读取 Endpoints 列表：

```http
GET /api/v1/namespaces/{namespace}/endpoints
```

读全部命名空间的 Endpoints 列表：

```http
GET /api/v1/endpoints
```

Watch Endpoints：

```http
GET /api/v1/watch/namespaces/{namespace}/endpoints/{name}
```

Watch Endpoints List:

```http
GET /api/v1/watch/namespaces/{namespace}/endpoints
```

Watch All Namespace Endpoints List:

```http
GET /api/v1/watch/endpoints
```



## Node

资源对象：

```go
type Node struct {
	metav1.TypeMeta `json:",inline"`
	// Standard object's metadata.
	// More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
	// +optional
	metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

	// Spec defines the behavior of a node.
	// https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
	// +optional
	Spec NodeSpec `json:"spec,omitempty" protobuf:"bytes,2,opt,name=spec"`

	// Most recently observed status of the node.
	// Populated by the system.
	// Read-only.
	// More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
	// +optional
	Status NodeStatus `json:"status,omitempty" protobuf:"bytes,3,opt,name=status"`
}
```

先来看 NodeSpec：

```go
type NodeSpec struct {
	// Pod 的子网，不过在使用了 Calico 之后，这里就没用了。
	PodCIDR string `json:"podCIDR,omitempty" protobuf:"bytes,1,opt,name=podCIDR"`

	// 多个 PodCIDR
	PodCIDRs []string `json:"podCIDRs,omitempty" protobuf:"bytes,7,opt,name=podCIDRs" patchStrategy:"merge"`

	// 用于云厂商
	ProviderID string `json:"providerID,omitempty" protobuf:"bytes,3,opt,name=providerID"`
	// 不调度，不让Pod调度到这个node上，默认为调度
	Unschedulable bool `json:"unschedulable,omitempty" protobuf:"varint,4,opt,name=unschedulable"`
	// node 的非亲和性
	// +optional
	Taints []Taint `json:"taints,omitempty" protobuf:"bytes,5,opt,name=taints"`
	// If specified, the source to get node configuration from
	// The DynamicKubeletConfig feature gate must be enabled for the Kubelet to use this field
	// +optional
	ConfigSource *NodeConfigSource `json:"configSource,omitempty" protobuf:"bytes,6,opt,name=configSource"`

	// Deprecated. Not all kubelets will set this field. Remove field after 1.13.
	// see: https://issues.k8s.io/61966
	// +optional
	DoNotUseExternalID string `json:"externalID,omitempty" protobuf:"bytes,2,opt,name=externalID"`
}
```

然后再来看 NodeStatus：

```go
type NodeStatus struct {
	// 节点信息，包括 CPU，内存，磁盘，最大 Pod 数量等。
	Capacity ResourceList `json:"capacity,omitempty" protobuf:"bytes,1,rep,name=capacity,casttype=ResourceList,castkey=ResourceName"`
	// 同上
	// Defaults to Capacity.
	// +optional
	Allocatable ResourceList `json:"allocatable,omitempty" protobuf:"bytes,2,rep,name=allocatable,casttype=ResourceList,castkey=ResourceName"`
	// 节点状态
	// More info: https://kubernetes.io/docs/concepts/nodes/node/#phase
	// 过期了
	// +optional
	Phase NodePhase `json:"phase,omitempty" protobuf:"bytes,3,opt,name=phase,casttype=NodePhase"`
	// 当前节点的一些信息，比如哪些 Pod 成功了，kublet 的状态等。
	Conditions []NodeCondition `json:"conditions,omitempty" patchStrategy:"merge" patchMergeKey:"type" protobuf:"bytes,4,rep,name=conditions"`
	// 节点的 IP 地址及主机名
	Addresses []NodeAddress `json:"addresses,omitempty" patchStrategy:"merge" patchMergeKey:"type" protobuf:"bytes,5,rep,name=addresses"`
	// kubelet 的端口，默认 10250
	DaemonEndpoints NodeDaemonEndpoints `json:"daemonEndpoints,omitempty" protobuf:"bytes,6,opt,name=daemonEndpoints"`
	// 节点的信息，比如 CPU 架构，各组件版本等。
	NodeInfo NodeSystemInfo `json:"nodeInfo,omitempty" protobuf:"bytes,7,opt,name=nodeInfo"`
	// 节点上的镜像列表
	Images []ContainerImage `json:"images,omitempty" protobuf:"bytes,8,rep,name=images"`
	// 节点上的正在使用卷列表
	VolumesInUse []UniqueVolumeName `json:"volumesInUse,omitempty" protobuf:"bytes,9,rep,name=volumesInUse"`
	// 节点上挂载的卷列表
	VolumesAttached []AttachedVolume `json:"volumesAttached,omitempty" protobuf:"bytes,10,rep,name=volumesAttached"`
	// Status of the config assigned to the node via the dynamic Kubelet config feature.
	// +optional
	Config *NodeConfigStatus `json:"config,omitempty" protobuf:"bytes,11,opt,name=config"`
}
```



下面来看 URL:

创建节点：

```http
POST /api/v1/nodes
```

修改节点：

```http
PATCH /api/v1/nodes/{name}
```

替换节点：

```http
PUT /api/v1/nodes/{name}
```

删除节点：

```http
DELETE /api/v1/nodes/{name}
```

删除一群节点：

```http
DELETE /api/v1/nodes
```

读取节点：

```http
GET /api/v1/nodes/{name}
```

读取节点列表：

```http
GET /api/v1/nodes
```

Watch 节点：

```
GET /api/v1/watch/nodes/{name}
```

Watch 一群节点：

```http
GET /api/v1/watch/nodes
```

修改节点状态：

```http
PATCH /api/v1/nodes/{name}/status
```

读取节点状态：

```http
GET /api/v1/nodes/{name}/status
```

替换节点状态：

```http
PUT /api/v1/nodes/{name}/status
```





## Binding

1.7 就过期了，不捣鼓了



## Event

先来看对象：

```go
type Event struct {
	metav1.TypeMeta `json:",inline"`
	// Standard object's metadata.
	// More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
	metav1.ObjectMeta `json:"metadata" protobuf:"bytes,1,opt,name=metadata"`

	// 目标对象的信息，比如 Endpoints，或 Service 等
	InvolvedObject ObjectReference `json:"involvedObject" protobuf:"bytes,2,opt,name=involvedObject"`

	// 原因
	Reason string `json:"reason,omitempty" protobuf:"bytes,3,opt,name=reason"`

	// 人类可读的详细信息
	Message string `json:"message,omitempty" protobuf:"bytes,4,opt,name=message"`

	// 事件源
	Source EventSource `json:"source,omitempty" protobuf:"bytes,5,opt,name=source"`

	// 第一次出现的时间
	FirstTimestamp metav1.Time `json:"firstTimestamp,omitempty" protobuf:"bytes,6,opt,name=firstTimestamp"`

	// 最后一次出现的时间
	LastTimestamp metav1.Time `json:"lastTimestamp,omitempty" protobuf:"bytes,7,opt,name=lastTimestamp"`

	// 数量
	Count int32 `json:"count,omitempty" protobuf:"varint,8,opt,name=count"`

	// 事件类型(Normal, Warning), new types could be added in the future
	// +optional
	Type string `json:"type,omitempty" protobuf:"bytes,9,opt,name=type"`

	// Time when this Event was first observed.
	// +optional
	EventTime metav1.MicroTime `json:"eventTime,omitempty" protobuf:"bytes,10,opt,name=eventTime"`

	// Data about the Event series this event represents or nil if it's a singleton Event.
	// +optional
	Series *EventSeries `json:"series,omitempty" protobuf:"bytes,11,opt,name=series"`

	// 动作
	Action string `json:"action,omitempty" protobuf:"bytes,12,opt,name=action"`

	// Optional secondary object for more complex actions.
	// +optional
	Related *ObjectReference `json:"related,omitempty" protobuf:"bytes,13,opt,name=related"`

	// 哪个组件的事件 e.g. `kubernetes.io/kubelet`.
	// +optional
	ReportingController string `json:"reportingComponent" protobuf:"bytes,14,opt,name=reportingComponent"`

	// 组件ID, e.g. `kubelet-xyzf`.
	// +optional
	ReportingInstance string `json:"reportingInstance" protobuf:"bytes,15,opt,name=reportingInstance"`
}
```

动作：

创建 Event：

```http
POST /api/v1/namespaces/{namespace}/events
```

修改 Event：

```http
PATCH /api/v1/namespaces/{namespace}/events/{name}
```

替换 Event：

```http
PUT /api/v1/namespaces/{namespace}/events/{name}
```

删除 Event：

```http
DELETE /api/v1/namespaces/{namespace}/events/{name}
```

删除一堆 Event：

```http
DELETE /api/v1/namespaces/{namespace}/events
```

读取 Event：

```http
GET /api/v1/namespaces/{namespace}/events/{name}
```

读取 Event 列表：

```http
GET /api/v1/namespaces/{namespace}/events
```

读取全部命名空间的 Event：

```http
GET /api/v1/events
```

Watch Event：

```http
GET /api/v1/watch/namespaces/{namespace}/events/{name}
```

Watch Event List:

```http
GET /api/v1/watch/namespaces/{namespace}/events
```

Watch 全部命名空间的 Event 列表：

````http
GET /api/v1/watch/events
````





## Namespace

对象结构：

```go
type Namespace struct {
	metav1.TypeMeta `json:",inline"`
	// Standard object's metadata.
	// More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
	// +optional
	metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

	// Spec defines the behavior of the Namespace.
	// More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
	// +optional
	Spec NamespaceSpec `json:"spec,omitempty" protobuf:"bytes,2,opt,name=spec"`

	// Status describes the current status of a Namespace.
	// More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
	// +optional
	Status NamespaceStatus `json:"status,omitempty" protobuf:"bytes,3,opt,name=status"`
}
```

NamespaceSpec:

```go
type NamespaceSpec struct {
  // 终结列表，必须为空才能被删除
	// Finalizers is an opaque list of values that must be empty to permanently remove object from storage.
	// More info: https://kubernetes.io/docs/tasks/administer-cluster/namespaces/
	// +optional
	Finalizers []FinalizerName `json:"finalizers,omitempty" protobuf:"bytes,1,rep,name=finalizers,casttype=FinalizerName"`
}
```

NamespaceStatus:

```go
type NamespaceStatus struct {
	// 状态，Active 或 Terminating
	Phase NamespacePhase `json:"phase,omitempty" protobuf:"bytes,1,opt,name=phase,casttype=NamespacePhase"`

	// Represents the latest available observations of a namespace's current state.
	// +optional
	// +patchMergeKey=type
	// +patchStrategy=merge
	Conditions []NamespaceCondition `json:"conditions,omitempty" patchStrategy:"merge" patchMergeKey:"type" protobuf:"bytes,2,rep,name=conditions"`
}
```

动作：

创建 Namespace：

```http
POST /api/v1/namespaces
```

修改 Namespace：

```http
PATCH /api/v1/namespaces/{name}
```

替换 Namespace：

```http
PUT /api/v1/namespaces/{name}
```

删除 Namespace：

```http
DELETE /api/v1/namespaces/{name}
```

读取 Namespace：

```http
GET /api/v1/namespaces/{name}
```

查看 Namespace List：

```http
GET /api/v1/namespaces
```

Watch Namespace:

```http
GET /api/v1/watch/namespaces/{name}
```

Watch Namespace List:

```http
GET /api/v1/watch/namespaces
```

修改 Namespace 状态：

```http
PATCH /api/v1/namespaces/{name}/status
```

读取 Namespace 状态：

```http
GET /api/v1/namespaces/{name}/status
```

替换 Namespace 状态：

```http
PUT /api/v1/namespaces/{name}/status
```





## Secret

资源对象：

```go
type Secret struct {
	metav1.TypeMeta `json:",inline"`
	// Standard object's metadata.
	// More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
	// +optional
	metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

	// 实际储存数据的地方，每个 Key 必须由字母数字字符、“-”、“ _”或“.”组成。
  // Value 是经过 BASE64 编码的人以字符串
	Data map[string][]byte `json:"data,omitempty" protobuf:"bytes,2,rep,name=data"`

  // 仅用于写，不会被读出。
	// stringData allows specifying non-binary secret data in string form.
	// It is provided as a write-only convenience method.
	// All keys and values are merged into the data field on write, overwriting any existing values.
	// It is never output when reading from the API.
	// +k8s:conversion-gen=false
	// +optional
	StringData map[string]string `json:"stringData,omitempty" protobuf:"bytes,4,rep,name=stringData"`

	// 数据的类型，
	// +optional
	Type SecretType `json:"type,omitempty" protobuf:"bytes,3,opt,name=type,casttype=SecretType"`
}
```

SecretType 可能的值如下：

```go
SecretTypeOpaque SecretType = "Opaque"
SecretTypeServiceAccountToken SecretType = "kubernetes.io/service-account-token"
SecretTypeDockercfg SecretType = "kubernetes.io/dockercfg"
SecretTypeDockerConfigJson SecretType = "kubernetes.io/dockerconfigjson"
SecretTypeBasicAuth SecretType = "kubernetes.io/basic-auth"
SecretTypeSSHAuth SecretType = "kubernetes.io/ssh-auth"
SecretTypeTLS SecretType = "kubernetes.io/tls"
SecretTypeBootstrapToken SecretType = "bootstrap.kubernetes.io/token"
```

操作：

创建 Secret：

```http
POST /api/v1/namespaces/{namespace}/secrets
```

修改 Secret：

```http
PATCH /api/v1/namespaces/{namespace}/secrets/{name}
```

替换 Secret：

```http
PUT /api/v1/namespaces/{namespace}/secrets/{name}
```

删除 Secret：

```http
DELETE /api/v1/namespaces/{namespace}/secrets/{name}
```

删除一堆 Secret：

```http
DELETE /api/v1/namespaces/{namespace}/secrets
```

读取 Secret：

```http
GET /api/v1/namespaces/{namespace}/secrets/{name}
```

读取 Secret List：

```http
GET /api/v1/namespaces/{namespace}/secrets
```

读取所有命名空间的 Secret List：

```http
GET /api/v1/secrets
```

Watch Secret:

```http
GET /api/v1/watch/namespaces/{namespace}/secrets/{name}
```

Watch Secret List:

```http
GET /api/v1/watch/namespaces/{namespace}/secrets
```

Watch All Namespace Secret List:

```http
GET /api/v1/watch/secrets
```





## ServiceAccount

对象结构：

```go
type ServiceAccount struct {
   metav1.TypeMeta `json:",inline"`
   // Standard object's metadata.
   // More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
   // +optional
   metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

   // Pod 运行过程中，允许 ServiceAccount 使用的 Secret 列表
   Secrets []ObjectReference `json:"secrets,omitempty" patchStrategy:"merge" patchMergeKey:"name" protobuf:"bytes,2,rep,name=secrets"`

   // 在 pull 镜像时用到的 Secrets
   ImagePullSecrets []LocalObjectReference `json:"imagePullSecrets,omitempty" protobuf:"bytes,3,rep,name=imagePullSecrets"`

   // AutomountServiceAccountToken指示作为该服务帐户运行的Pod是否应具有自动装入的API Token。
   // 可以在 Pod 上覆盖
   // +optional
   AutomountServiceAccountToken *bool `json:"automountServiceAccountToken,omitempty" protobuf:"varint,4,opt,name=automountServiceAccountToken"`
}
```

创建 ServiceAccount：

```http
POST /api/v1/namespaces/{namespace}/serviceaccounts
```

更新 ServiceAccount：

```http
PATCH /api/v1/namespaces/{namespace}/serviceaccounts/{name}
```

替换 ServiceAccount：

```http
PUT /api/v1/namespaces/{namespace}/serviceaccounts/{name}
```

删除 ServiceAccount：

```http
DELETE /api/v1/namespaces/{namespace}/serviceaccounts/{name}
```

删除一堆 ServiceAccount：

```http
DELETE /api/v1/namespaces/{namespace}/serviceaccounts
```

读取 ServiceAccount：

```http
GET /api/v1/namespaces/{namespace}/serviceaccounts/{name}
```

读取 ServiceAccount List：

```http
GET /api/v1/namespaces/{namespace}/serviceaccounts
```

读取所有命名空间的 ServiceAccount List：

```http
GET /api/v1/serviceaccounts
```

Watch ServiceAccount:

```http
GET /api/v1/watch/namespaces/{namespace}/serviceaccounts/{name}
```

Watch ServiceAccount List:

```http
GET /api/v1/watch/namespaces/{namespace}/serviceaccounts
```

Watch All Namespace ServiceAccount List:

```http
GET /api/v1/watch/serviceaccounts
```



## PersistentVolume

对象结构：

```go
type PersistentVolume struct {
   metav1.TypeMeta `json:",inline"`
   // Standard object's metadata.
   // More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
   // +optional
   metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

   // Spec defines a specification of a persistent volume owned by the cluster.
   // Provisioned by an administrator.
   // More info: https://kubernetes.io/docs/concepts/storage/persistent-volumes#persistent-volumes
   // +optional
   Spec PersistentVolumeSpec `json:"spec,omitempty" protobuf:"bytes,2,opt,name=spec"`

   // Status represents the current information/status for the persistent volume.
   // Populated by the system.
   // Read-only.
   // More info: https://kubernetes.io/docs/concepts/storage/persistent-volumes#persistent-volumes
   // +optional
   Status PersistentVolumeStatus `json:"status,omitempty" protobuf:"bytes,3,opt,name=status"`
}
```



PersistentVolumeSpec：

```go
type PersistentVolumeSpec struct {
	// 资源的描述，
	Capacity ResourceList `json:"capacity,omitempty" protobuf:"bytes,1,rep,name=capacity,casttype=ResourceList,castkey=ResourceName"`
	// The actual volume backing the persistent volume.
	PersistentVolumeSource `json:",inline" protobuf:"bytes,2,opt,name=persistentVolumeSource"`
	// 访问模式，包括 ReadWriteOnce、ReadOnlyMany、ReadWriteMany
	AccessModes []PersistentVolumeAccessMode `json:"accessModes,omitempty" protobuf:"bytes,3,rep,name=accessModes,casttype=PersistentVolumeAccessMode"`
	// ClaimRef是PersistentVolume和PersistentVolumeClaim之间双向绑定的一部分。
	// 绑定时不应为 空
	// claim.VolumeName 是 PV 和 PVC 之间的认证
	// More info: https://kubernetes.io/docs/concepts/storage/persistent-volumes#binding
	// +optional
	ClaimRef *ObjectReference `json:"claimRef,omitempty" protobuf:"bytes,4,opt,name=claimRef"`
	// 当一个持久卷从声明中释放时会发生什么。
  // 有效选项包括Retain（手动创建的PersistentVolumes的默认值）、Delete（动态配置的PersistentVolumes的默认值）和Recycle（不推荐）。
  //此PersistentVolume的基础卷插件必须支持回收。
	PersistentVolumeReclaimPolicy PersistentVolumeReclaimPolicy `json:"persistentVolumeReclaimPolicy,omitempty" protobuf:"bytes,5,opt,name=persistentVolumeReclaimPolicy,casttype=PersistentVolumeReclaimPolicy"`
	// StorageClassName 的名字. 为空意味着 卷 不属于任何 StorageClass
	StorageClassName string `json:"storageClassName,omitempty" protobuf:"bytes,6,opt,name=storageClassName"`
	// 一个挂在选项列表, 例如 ["ro", "soft"]. 未验证-如果其中一个无效，装载将失败。比如CephFS，RBD等。
	// More info: https://kubernetes.io/docs/concepts/storage/persistent-volumes/#mount-options
	// +optional
	MountOptions []string `json:"mountOptions,omitempty" protobuf:"bytes,7,opt,name=mountOptions"`
	// volumeMode定义卷是要与格式化的文件系统一起使用还是保持原始块状态。文件系统的值在未包含在规范中时是隐含的。
  // 取值为 “Block” 或 “Filesystem”
	// This is a beta feature.
	VolumeMode *PersistentVolumeMode `json:"volumeMode,omitempty" protobuf:"bytes,8,opt,name=volumeMode,casttype=PersistentVolumeMode"`
	// 节点亲和性
	NodeAffinity *VolumeNodeAffinity `json:"nodeAffinity,omitempty" protobuf:"bytes,9,opt,name=nodeAffinity"`
}
```

PersistentVolumeStatus：

```go
type PersistentVolumeStatus struct {
	// 指示卷是否可用、是否绑定到声明或是否由声明释放。可能的值包括：Pending、Available、Bound、Released、Failed
	Phase PersistentVolumePhase `json:"phase,omitempty" protobuf:"bytes,1,opt,name=phase,casttype=PersistentVolumePhase"`
	// 一个人类可读的信息包含为什么会处在这个状态
	Message string `json:"message,omitempty" protobuf:"bytes,2,opt,name=message"`
	// 一个简短的字符串，描述了原因，会在 cli 工具显示。
	Reason string `json:"reason,omitempty" protobuf:"bytes,3,opt,name=reason"`
}
```

创建 PV：

```http
POST /api/v1/persistentvolumes
```

修改 PV：

```http
PATCH /api/v1/persistentvolumes/{name}
```

替换 PV：

```http
PUT /api/v1/persistentvolumes/{name}
```

删除 PV：

```http
DELETE /api/v1/persistentvolumes/{name}
```

删除一堆PV：

```http
DELETE /api/v1/persistentvolumes
```

读取 PV：

```http
GET /api/v1/persistentvolumes/{name}
```

读取 PV List：

```http
GET /api/v1/persistentvolumes
```

Watch PV:

```http
GET /api/v1/watch/persistentvolumes/{name}
```

Watch PV List:

```http
GET /api/v1/watch/persistentvolumes
```

修改 PV 状态：

```http
PATCH /api/v1/persistentvolumes/{name}/status
```

读取 PV 状态：

````http
GET /api/v1/persistentvolumes/{name}/status
````

替换 PV 状态：

```http
PUT /api/v1/persistentvolumes/{name}/status
```





## PersistentVolumeClaim

对象结构：

```go
type PersistentVolumeClaim struct {
	metav1.TypeMeta `json:",inline"`
	// Standard object's metadata.
	// More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
	// +optional
	metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

	// Spec defines the desired characteristics of a volume requested by a pod author.
	// More info: https://kubernetes.io/docs/concepts/storage/persistent-volumes#persistentvolumeclaims
	// +optional
	Spec PersistentVolumeClaimSpec `json:"spec,omitempty" protobuf:"bytes,2,opt,name=spec"`

	// Status represents the current information/status of a persistent volume claim.
	// Read-only.
	// More info: https://kubernetes.io/docs/concepts/storage/persistent-volumes#persistentvolumeclaims
	// +optional
	Status PersistentVolumeClaimStatus `json:"status,omitempty" protobuf:"bytes,3,opt,name=status"`
}
```

PersistentVolumeClaimSpec:

```go
type PersistentVolumeClaimSpec struct {
	// 访问模式，包括 ReadWriteOnce、ReadOnlyMany、ReadWriteMany
	AccessModes []PersistentVolumeAccessMode `json:"accessModes,omitempty" protobuf:"bytes,1,rep,name=accessModes,casttype=PersistentVolumeAccessMode"`
	// 通过 Label 选择卷
	Selector *metav1.LabelSelector `json:"selector,omitempty" protobuf:"bytes,4,opt,name=selector"`
	// 资源表示卷应具有的最小资源。
	// More info: https://kubernetes.io/docs/concepts/storage/persistent-volumes#resources
	// +optional
	Resources ResourceRequirements `json:"resources,omitempty" protobuf:"bytes,2,opt,name=resources"`
	// 对声明的PersistentVolume的绑定引用。
	// +optional
	VolumeName string `json:"volumeName,omitempty" protobuf:"bytes,3,opt,name=volumeName"`
	// 动态 PV，StorageClass 的名称
	// More info: https://kubernetes.io/docs/concepts/storage/persistent-volumes#class-1
	// +optional
	StorageClassName *string `json:"storageClassName,omitempty" protobuf:"bytes,5,opt,name=storageClassName"`
	// volumeMode定义卷是要与格式化的文件系统一起使用还是保持原始块状态。文件系统的值在未包含在规范中时是隐含的。
  // 取值为 “Block” 或 “Filesystem”
	// This is a beta feature.
	// +optional
	VolumeMode *PersistentVolumeMode `json:"volumeMode,omitempty" protobuf:"bytes,6,opt,name=volumeMode,casttype=PersistentVolumeMode"`
	// 此字段要求启用VolumeSnapshotDataSource alpha功能门，并且当前VolumeSnapshot是唯一受支持的数据源。
  // 如果provisioner可以支持VolumeSnapshot数据源，它将创建一个新卷，同时将数据还原到该卷。
  // 如果provisioner不支持VolumeSnapshot数据源，则不会创建卷，并将失败报告为事件。
  // 在将来，我们计划支持更多的数据源类型，并且provisioner的行为可能会改变。
	DataSource *TypedLocalObjectReference `json:"dataSource,omitempty" protobuf:"bytes,7,opt,name=dataSource"`
}
```

PersistentVolumeClaimStatus:

```go
type PersistentVolumeClaimStatus struct {
	// 状态，Pending、Bound、Lost
	Phase PersistentVolumeClaimPhase `json:"phase,omitempty" protobuf:"bytes,1,opt,name=phase,casttype=PersistentVolumeClaimPhase"`
	// 访问模式，包括 ReadWriteOnce、ReadOnlyMany、ReadWriteMany
	AccessModes []PersistentVolumeAccessMode `json:"accessModes,omitempty" protobuf:"bytes,2,rep,name=accessModes,casttype=PersistentVolumeAccessMode"`
	// 表示基础卷的实际资源。
	Capacity ResourceList `json:"capacity,omitempty" protobuf:"bytes,3,rep,name=capacity,casttype=ResourceList,castkey=ResourceName"`
	// 持久卷声明的当前状态。如果正在调整基础持久卷的大小，则条件将设置为“ResizeStarted”。
	Conditions []PersistentVolumeClaimCondition `json:"conditions,omitempty" patchStrategy:"merge" patchMergeKey:"type" protobuf:"bytes,4,rep,name=conditions"`
}
```

创建 PVC：

```http
POST /api/v1/namespaces/{namespace}/persistentvolumeclaims
```

修改 PVC：

```http
PATCH /api/v1/namespaces/{namespace}/persistentvolumeclaims/{name}
```

替换 PVC：

```http
PATCH /api/v1/namespaces/{namespace}/persistentvolumeclaims/{name}
```

删除 PVC：

```http
DELETE /api/v1/namespaces/{namespace}/persistentvolumeclaims/{name}
```

删除一堆 PVC：

```http
DELETE /api/v1/namespaces/{namespace}/persistentvolumeclaims
```

读取 PVC：

```http
GET /api/v1/namespaces/{namespace}/persistentvolumeclaims/{name}
```

读取 PVC List：

```http
GET /api/v1/namespaces/{namespace}/persistentvolumeclaims
```

读取全部命名空间的 PVC：

```http
GET /api/v1/persistentvolumeclaims
```

Watch PVC:

```http
GET /api/v1/watch/namespaces/{namespace}/persistentvolumeclaims/{name}
```

Watch PVC List:

```http
GET /api/v1/watch/namespaces/{namespace}/persistentvolumeclaims
```

Watch All Namespace PVC List:

```http
GET /api/v1/watch/persistentvolumeclaims
```

修改 PVC 状态：

```http
PATCH /api/v1/namespaces/{namespace}/persistentvolumeclaims/{name}/status
```

读取 PVC 状态：

```http
GET /api/v1/namespaces/{namespace}/persistentvolumeclaims/{name}/status
```

替换 PVC 状态：

```http
PUT /api/v1/namespaces/{namespace}/persistentvolumeclaims/{name}/status
```



## ComponentStatus

对象结构：

```go
type ComponentStatus struct {
	metav1.TypeMeta `json:",inline"`
	// Standard object's metadata.
	// More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
	// +optional
	metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

	// 观察到的组件状态列表
	// +optional
	// +patchMergeKey=type
	// +patchStrategy=merge
	Conditions []ComponentCondition `json:"conditions,omitempty" patchStrategy:"merge" patchMergeKey:"type" protobuf:"bytes,2,rep,name=conditions"`
}
```

读取 ：

```http
GET /api/v1/componentstatuses/{name}
```

读取列表：

```http
GET /api/v1/componentstatuses
```



## ConfigMap

对象结构：

```go
type ConfigMap struct {
	metav1.TypeMeta `json:",inline"`
	// Standard object's metadata.
	// More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
	// +optional
	metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

	// key-value 键值对 MAP
	Data map[string]string `json:"data,omitempty" protobuf:"bytes,2,rep,name=data"`

	// 二进制数据
	BinaryData map[string][]byte `json:"binaryData,omitempty" protobuf:"bytes,3,rep,name=binaryData"`
}
```

创建：

```http
POST /api/v1/namespaces/{namespace}/configmaps
```

修改：

```http
PATCH /api/v1/namespaces/{namespace}/configmaps/{name}
```

替换：

```http
PUT /api/v1/namespaces/{namespace}/configmaps/{name}
```

删除：

```http
DELETE /api/v1/namespaces/{namespace}/configmaps/{name}
```

删除一堆：

```http
DELETE /api/v1/namespaces/{namespace}/configmaps
```

读取 ：

```http
GET /api/v1/namespaces/{namespace}/configmaps/{name}
```

读取列表：

```http
GET /api/v1/namespaces/{namespace}/configmaps
```

读取全部命名空间的列表：

```http
GET /api/v1/configmaps
```



## LimitRange

`LimitRange`是在`pod`和`container`级别的资源限制

对象结构：

```go
type LimitRange struct {
   metav1.TypeMeta `json:",inline"`
   // Standard object's metadata.
   // More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
   // +optional
   metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

   // Spec defines the limits enforced.
   // More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
   // +optional
   Spec LimitRangeSpec `json:"spec,omitempty" protobuf:"bytes,2,opt,name=spec"`
}
```

LimitRangeSpec:

```go
type LimitRangeSpec struct {
	// Limits is the list of LimitRangeItem objects that are enforced.
	Limits []LimitRangeItem `json:"limits" protobuf:"bytes,1,rep,name=limits"`
}
```

LimitRangeItem:

```go
type LimitRangeItem struct {
   // Type of resource that this limit applies to.
   // +optional
   Type LimitType `json:"type,omitempty" protobuf:"bytes,1,opt,name=type,casttype=LimitType"`
   // Max usage constraints on this kind by resource name.
   // +optional
   Max ResourceList `json:"max,omitempty" protobuf:"bytes,2,rep,name=max,casttype=ResourceList,castkey=ResourceName"`
   // Min usage constraints on this kind by resource name.
   // +optional
   Min ResourceList `json:"min,omitempty" protobuf:"bytes,3,rep,name=min,casttype=ResourceList,castkey=ResourceName"`
   // Default resource requirement limit value by resource name if resource limit is omitted.
   // +optional
   Default ResourceList `json:"default,omitempty" protobuf:"bytes,4,rep,name=default,casttype=ResourceList,castkey=ResourceName"`
   // DefaultRequest is the default resource requirement request value by resource name if resource request is omitted.
   // +optional
   DefaultRequest ResourceList `json:"defaultRequest,omitempty" protobuf:"bytes,5,rep,name=defaultRequest,casttype=ResourceList,castkey=ResourceName"`
   // MaxLimitRequestRatio if specified, the named resource must have a request and limit that are both non-zero where limit divided by request is less than or equal to the enumerated value; this represents the max burst for the named resource.
   // +optional
   MaxLimitRequestRatio ResourceList `json:"maxLimitRequestRatio,omitempty" protobuf:"bytes,6,rep,name=maxLimitRequestRatio,casttype=ResourceList,castkey=ResourceName"`
}
```

创建：

```http
POST /api/v1/namespaces/{namespace}/limitranges
```

修改：

```http
PATCH /api/v1/namespaces/{namespace}/limitranges/{name}
```

替换：

```http
PUT /api/v1/namespaces/{namespace}/limitranges/{name}
```

删除：

```http
DELETE /api/v1/namespaces/{namespace}/limitranges/{name}
```

删除一堆：

```http
DELETE /api/v1/namespaces/{namespace}/limitranges
```

读取：

```http
GET /api/v1/namespaces/{namespace}/limitranges/{name}
```

读取列表：

```http
GET /api/v1/namespaces/{namespace}/limitranges
```

读取全部命名空间的列表：

```http
GET /api/v1/limitranges
```



## ResourceQuota

资源配额

对象结构：

```go
type ResourceQuota struct {
	metav1.TypeMeta `json:",inline"`
	// Standard object's metadata.
	// More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
	// +optional
	metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

	// Spec defines the desired quota.
	// https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
	// +optional
	Spec ResourceQuotaSpec `json:"spec,omitempty" protobuf:"bytes,2,opt,name=spec"`

	// Status defines the actual enforced quota and its current usage.
	// https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
	// +optional
	Status ResourceQuotaStatus `json:"status,omitempty" protobuf:"bytes,3,opt,name=status"`
}
```

ResourceQuotaSpec:

```go
type ResourceQuotaSpec struct {
	// 资源列表
	Hard ResourceList `json:"hard,omitempty" protobuf:"bytes,1,rep,name=hard,casttype=ResourceList,castkey=ResourceName"`
	// 必须与配额跟踪的每个对象匹配的筛选器的集合。
  // 如果未指定，配额将匹配所有对象。
	// +optional
	Scopes []ResourceQuotaScope `json:"scopes,omitempty" protobuf:"bytes,2,rep,name=scopes,casttype=ResourceQuotaScope"`
	// scopeSelector也是一组类似于作用域的筛选器，这些作用域必须与配额跟踪的每个对象匹配，但使用scopeSelector运算符和可能的值组合来表示。
  // 要匹配资源，必须同时匹配作用域和作用域选择器（如果在规范中指定）。
	ScopeSelector *ScopeSelector `json:"scopeSelector,omitempty" protobuf:"bytes,3,opt,name=scopeSelector"`
}
```

ResourceQuotaStatus：

```go
type ResourceQuotaStatus struct {
   // 资源列表
   Hard ResourceList `json:"hard,omitempty" protobuf:"bytes,1,rep,name=hard,casttype=ResourceList,castkey=ResourceName"`
   // 已使用的资源列表
   Used ResourceList `json:"used,omitempty" protobuf:"bytes,2,rep,name=used,casttype=ResourceList,castkey=ResourceName"`
}
```

创建：

```http
POST /api/v1/namespaces/{namespace}/resourcequotas
```

修改：

```http
PATCH /api/v1/namespaces/{namespace}/resourcequotas/{name}
```

替换：

```http
PUT /api/v1/namespaces/{namespace}/resourcequotas/{name}
```

删除：

```http
DELETE /api/v1/namespaces/{namespace}/resourcequotas/{name}
```

删除一堆：

```http
DELETE /api/v1/namespaces/{namespace}/resourcequotas/{name}
```

读取：

```http
GET /api/v1/namespaces/{namespace}/resourcequotas/{name}
```

读取列表：

```http
GET /api/v1/namespaces/{namespace}/resourcequotas
```

读取所有命名空间的列表：

```http
GET /api/v1/resourcequotas
```

修改状态：

```http
PATCH /api/v1/namespaces/{namespace}/resourcequotas/{name}/status
```

读取状态：

```http
GET /api/v1/namespaces/{namespace}/resourcequotas/{name}/status
```

替换状态：

```http
PUT /api/v1/namespaces/{namespace}/resourcequotas/{name}/status
```



## ReplicationController

Replication Controller 被 ReplicaSet 和 Deployment 按在地上摩擦，不捣鼓了。









































