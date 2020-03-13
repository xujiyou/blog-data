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

```
GET /api/v1
```



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
	// If specified, indicates the pod's priority. "system-node-critical" and
	// "system-cluster-critical" are two special keywords which indicate the
	// highest priorities with the former being the highest priority. Any other
	// name must be defined by creating a PriorityClass object with that name.
	// If not specified, the pod priority will be default or zero if there is no
	// default.
	// +optional
	PriorityClassName string `json:"priorityClassName,omitempty" protobuf:"bytes,24,opt,name=priorityClassName"`
	// The priority value. Various system components use this field to find the
	// priority of the pod. When Priority Admission Controller is enabled, it
	// prevents users from setting this field. The admission controller populates
	// this field from PriorityClassName.
	// The higher the value, the higher the priority.
	// +optional
	Priority *int32 `json:"priority,omitempty" protobuf:"bytes,25,opt,name=priority"`
	// Specifies the DNS parameters of a pod.
	// Parameters specified here will be merged to the generated DNS
	// configuration based on DNSPolicy.
	// +optional
	DNSConfig *PodDNSConfig `json:"dnsConfig,omitempty" protobuf:"bytes,26,opt,name=dnsConfig"`
	// If specified, all readiness gates will be evaluated for pod readiness.
	// A pod is ready when all its containers are ready AND
	// all conditions specified in the readiness gates have status equal to "True"
	// More info: https://git.k8s.io/enhancements/keps/sig-network/0007-pod-ready%2B%2B.md
	// +optional
	ReadinessGates []PodReadinessGate `json:"readinessGates,omitempty" protobuf:"bytes,28,opt,name=readinessGates"`
	// RuntimeClassName refers to a RuntimeClass object in the node.k8s.io group, which should be used
	// to run this pod.  If no RuntimeClass resource matches the named class, the pod will not be run.
	// If unset or empty, the "legacy" RuntimeClass will be used, which is an implicit class with an
	// empty definition that uses the default runtime handler.
	// More info: https://git.k8s.io/enhancements/keps/sig-node/runtime-class.md
	// This is a beta feature as of Kubernetes v1.14.
	// +optional
	RuntimeClassName *string `json:"runtimeClassName,omitempty" protobuf:"bytes,29,opt,name=runtimeClassName"`
	// EnableServiceLinks indicates whether information about services should be injected into pod's
	// environment variables, matching the syntax of Docker links.
	// Optional: Defaults to true.
	// +optional
	EnableServiceLinks *bool `json:"enableServiceLinks,omitempty" protobuf:"varint,30,opt,name=enableServiceLinks"`
	// PreemptionPolicy is the Policy for preempting pods with lower priority.
	// One of Never, PreemptLowerPriority.
	// Defaults to PreemptLowerPriority if unset.
	// This field is alpha-level and is only honored by servers that enable the NonPreemptingPriority feature.
	// +optional
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











