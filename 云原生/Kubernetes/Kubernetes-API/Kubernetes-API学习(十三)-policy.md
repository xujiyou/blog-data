# Kubernetes API 学习(十三) - policy

```
$ kubectl api-resources --api-group=policy
NAME                   SHORTNAMES   APIGROUP   NAMESPACED   KIND
poddisruptionbudgets   pdb          policy     true         PodDisruptionBudget
podsecuritypolicies    psp          policy     false        PodSecurityPolicy
```

源码位于 API 源码包的 policy 包中。



## PodDisruptionBudget

在Kubernetes中，为了保证业务不中断或业务SLA不降级，需要将应用进行集群化部署。通过PodDisruptionBudget控制器可以设置应用POD集群处于运行状态最低个数，也可以设置应用POD集群处于运行状态的最低百分比，这样可以保证在主动销毁应用POD的时候，不会一次性销毁太多的应用[POD](https://www.kubernetes.org.cn/kubernetes-pod)，从而保证业务不中断或业务SLA不降级。

在[Kubernetes 1.5](https://www.kubernetes.org.cn/tags/kubernetes1-5)中，kubectl drain命令已经支持了PodDisruptionBudget控制器，在进行kubectl drain操作时会根据PodDisruptionBudget控制器判断应用POD集群数量，进而保证在业务不中断或业务SLA不降级的情况下进行应用POD销毁。

对象结构：

```go
type PodDisruptionBudget struct {
	metav1.TypeMeta `json:",inline"`
	// +optional
	metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

	// Specification of the desired behavior of the PodDisruptionBudget.
	// +optional
	Spec PodDisruptionBudgetSpec `json:"spec,omitempty" protobuf:"bytes,2,opt,name=spec"`
	// Most recently observed status of the PodDisruptionBudget.
	// +optional
	Status PodDisruptionBudgetStatus `json:"status,omitempty" protobuf:"bytes,3,opt,name=status"`
}
```

PodDisruptionBudgetSpec:

```go
type PodDisruptionBudgetSpec struct {
   // 最小获得数
   MinAvailable *intstr.IntOrString `json:"minAvailable,omitempty" protobuf:"bytes,1,opt,name=minAvailable"`

   // label 选择器
   Selector *metav1.LabelSelector `json:"selector,omitempty" protobuf:"bytes,2,opt,name=selector"`

   // 最大可获得数
   MaxUnavailable *intstr.IntOrString `json:"maxUnavailable,omitempty" protobuf:"bytes,3,opt,name=maxUnavailable"`
}
```

PodDisruptionBudgetStatus:

```go
type PodDisruptionBudgetStatus struct {
   // 更新此PDB状态时观察到的最新一代。 仅当observedGeneration等于PDB的对象生成时，PodDisruptionsAllowed和其他状态信息才有效。
   // +optional
   ObservedGeneration int64 `json:"observedGeneration,omitempty" protobuf:"varint,1,opt,name=observedGeneration"`

   // 中断的 Pod
   // DisruptedPods contains information about pods whose eviction was
   // processed by the API server eviction subresource handler but has not
   // yet been observed by the PodDisruptionBudget controller.
   // A pod will be in this map from the time when the API server processed the
   // eviction request to the time when the pod is seen by PDB controller
   // as having been marked for deletion (or after a timeout). The key in the map is the name of the pod
   // and the value is the time when the API server processed the eviction request. If
   // the deletion didn't occur and a pod is still there it will be removed from
   // the list automatically by PodDisruptionBudget controller after some time.
   // If everything goes smooth this map should be empty for the most of the time.
   // Large number of entries in the map may indicate problems with pod deletions.
   // +optional
   DisruptedPods map[string]metav1.Time `json:"disruptedPods,omitempty" protobuf:"bytes,2,rep,name=disruptedPods"`

   // 允许的 Pod 中断的数量
   PodDisruptionsAllowed int32 `json:"disruptionsAllowed" protobuf:"varint,3,opt,name=disruptionsAllowed"`

   // 当前健康 Pod 的数量
   CurrentHealthy int32 `json:"currentHealthy" protobuf:"varint,4,opt,name=currentHealthy"`

   // 最小的期望健康的数量
   DesiredHealthy int32 `json:"desiredHealthy" protobuf:"varint,5,opt,name=desiredHealthy"`

   // Pod 总数
   ExpectedPods int32 `json:"expectedPods" protobuf:"varint,6,opt,name=expectedPods"`
}
```

创建：

```http
POST /apis/policy/v1beta1/namespaces/{namespace}/poddisruptionbudgets
```

添加配置：

```http
PATCH /apis/policy/v1beta1/namespaces/{namespace}/poddisruptionbudgets/{name}
```

修改：

```http
PUT /apis/policy/v1beta1/namespaces/{namespace}/poddisruptionbudgets/{name}
```

删除：

```http
DELETE /apis/policy/v1beta1/namespaces/{namespace}/poddisruptionbudgets/{name}
```

删除一堆：

```http
DELETE /apis/policy/v1beta1/namespaces/{namespace}/poddisruptionbudgets
```

读取：

```http
GET /apis/policy/v1beta1/namespaces/{namespace}/poddisruptionbudgets/{name}
```

读取列表：

```http
GET /apis/policy/v1beta1/namespaces/{namespace}/poddisruptionbudgets
```

读取全部命名空间的列表：

```http
GET /apis/policy/v1beta1/poddisruptionbudgets
```

添加状态：

```http
PATCH /apis/policy/v1beta1/namespaces/{namespace}/poddisruptionbudgets/{name}/status
```

读取状态：

```http
GET /apis/policy/v1beta1/namespaces/{namespace}/poddisruptionbudgets/{name}/status
```

修改状态：

```http
PUT /apis/policy/v1beta1/namespaces/{namespace}/poddisruptionbudgets/{name}/status
```





## PodSecurityPolicy

查看介绍：https://kubernetes.io/zh/docs/concepts/policy/pod-security-policy/

官方介绍看的模糊不清，这篇比较详细：https://www.qikqiak.com/post/setup-psp-in-k8s/

PodSecurityPolicy 和 RDBC 是有关联的。可以关联到 ClusterRole 之上。

对象结构：

```go
type PodSecurityPolicy struct {
   metav1.TypeMeta `json:",inline"`
   // Standard object's metadata.
   // More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
   // +optional
   metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

   // spec defines the policy enforced.
   // +optional
   Spec PodSecurityPolicySpec `json:"spec,omitempty" protobuf:"bytes,2,opt,name=spec"`
}
```

PodSecurityPolicySpec:

```go
type PodSecurityPolicySpec struct {
   // 是否可以运行在特权模式
   Privileged bool `json:"privileged,omitempty" protobuf:"varint,1,opt,name=privileged"`
   // defaultAddCapabilities is the default set of capabilities that will be added to the container
   // unless the pod spec specifically drops the capability.  You may not list a capability in both
   // defaultAddCapabilities and requiredDropCapabilities. Capabilities added here are implicitly
   // allowed, and need not be included in the allowedCapabilities list.
   // +optional
   DefaultAddCapabilities []v1.Capability `json:"defaultAddCapabilities,omitempty" protobuf:"bytes,2,rep,name=defaultAddCapabilities,casttype=k8s.io/api/core/v1.Capability"`
   // requiredDropCapabilities are the capabilities that will be dropped from the container.  These
   // are required to be dropped and cannot be added.
   // +optional
   RequiredDropCapabilities []v1.Capability `json:"requiredDropCapabilities,omitempty" protobuf:"bytes,3,rep,name=requiredDropCapabilities,casttype=k8s.io/api/core/v1.Capability"`
   // allowedCapabilities is a list of capabilities that can be requested to add to the container.
   // Capabilities in this field may be added at the pod author's discretion.
   // You must not list a capability in both allowedCapabilities and requiredDropCapabilities.
   // +optional
   AllowedCapabilities []v1.Capability `json:"allowedCapabilities,omitempty" protobuf:"bytes,4,rep,name=allowedCapabilities,casttype=k8s.io/api/core/v1.Capability"`
   // volumes is a white list of allowed volume plugins. Empty indicates that
   // no volumes may be used. To allow all volumes you may use '*'.
   // +optional
   Volumes []FSType `json:"volumes,omitempty" protobuf:"bytes,5,rep,name=volumes,casttype=FSType"`
   // hostNetwork determines if the policy allows the use of HostNetwork in the pod spec.
   // +optional
   HostNetwork bool `json:"hostNetwork,omitempty" protobuf:"varint,6,opt,name=hostNetwork"`
   // hostPorts determines which host port ranges are allowed to be exposed.
   // +optional
   HostPorts []HostPortRange `json:"hostPorts,omitempty" protobuf:"bytes,7,rep,name=hostPorts"`
   // hostPID determines if the policy allows the use of HostPID in the pod spec.
   // +optional
   HostPID bool `json:"hostPID,omitempty" protobuf:"varint,8,opt,name=hostPID"`
   // hostIPC determines if the policy allows the use of HostIPC in the pod spec.
   // +optional
   HostIPC bool `json:"hostIPC,omitempty" protobuf:"varint,9,opt,name=hostIPC"`
   // seLinux is the strategy that will dictate the allowable labels that may be set.
   SELinux SELinuxStrategyOptions `json:"seLinux" protobuf:"bytes,10,opt,name=seLinux"`
   // runAsUser is the strategy that will dictate the allowable RunAsUser values that may be set.
   RunAsUser RunAsUserStrategyOptions `json:"runAsUser" protobuf:"bytes,11,opt,name=runAsUser"`
   // RunAsGroup is the strategy that will dictate the allowable RunAsGroup values that may be set.
   // If this field is omitted, the pod's RunAsGroup can take any value. This field requires the
   // RunAsGroup feature gate to be enabled.
   // +optional
   RunAsGroup *RunAsGroupStrategyOptions `json:"runAsGroup,omitempty" protobuf:"bytes,22,opt,name=runAsGroup"`
   // supplementalGroups is the strategy that will dictate what supplemental groups are used by the SecurityContext.
   SupplementalGroups SupplementalGroupsStrategyOptions `json:"supplementalGroups" protobuf:"bytes,12,opt,name=supplementalGroups"`
   // fsGroup is the strategy that will dictate what fs group is used by the SecurityContext.
   FSGroup FSGroupStrategyOptions `json:"fsGroup" protobuf:"bytes,13,opt,name=fsGroup"`
   // readOnlyRootFilesystem when set to true will force containers to run with a read only root file
   // system.  If the container specifically requests to run with a non-read only root file system
   // the PSP should deny the pod.
   // If set to false the container may run with a read only root file system if it wishes but it
   // will not be forced to.
   // +optional
   ReadOnlyRootFilesystem bool `json:"readOnlyRootFilesystem,omitempty" protobuf:"varint,14,opt,name=readOnlyRootFilesystem"`
   // defaultAllowPrivilegeEscalation controls the default setting for whether a
   // process can gain more privileges than its parent process.
   // +optional
   DefaultAllowPrivilegeEscalation *bool `json:"defaultAllowPrivilegeEscalation,omitempty" protobuf:"varint,15,opt,name=defaultAllowPrivilegeEscalation"`
   // allowPrivilegeEscalation determines if a pod can request to allow
   // privilege escalation. If unspecified, defaults to true.
   // +optional
   AllowPrivilegeEscalation *bool `json:"allowPrivilegeEscalation,omitempty" protobuf:"varint,16,opt,name=allowPrivilegeEscalation"`
   // allowedHostPaths is a white list of allowed host paths. Empty indicates
   // that all host paths may be used.
   // +optional
   AllowedHostPaths []AllowedHostPath `json:"allowedHostPaths,omitempty" protobuf:"bytes,17,rep,name=allowedHostPaths"`
   // allowedFlexVolumes is a whitelist of allowed Flexvolumes.  Empty or nil indicates that all
   // Flexvolumes may be used.  This parameter is effective only when the usage of the Flexvolumes
   // is allowed in the "volumes" field.
   // +optional
   AllowedFlexVolumes []AllowedFlexVolume `json:"allowedFlexVolumes,omitempty" protobuf:"bytes,18,rep,name=allowedFlexVolumes"`
   // AllowedCSIDrivers is a whitelist of inline CSI drivers that must be explicitly set to be embedded within a pod spec.
   // An empty value indicates that any CSI driver can be used for inline ephemeral volumes.
   // This is an alpha field, and is only honored if the API server enables the CSIInlineVolume feature gate.
   // +optional
   AllowedCSIDrivers []AllowedCSIDriver `json:"allowedCSIDrivers,omitempty" protobuf:"bytes,23,rep,name=allowedCSIDrivers"`
   // allowedUnsafeSysctls is a list of explicitly allowed unsafe sysctls, defaults to none.
   // Each entry is either a plain sysctl name or ends in "*" in which case it is considered
   // as a prefix of allowed sysctls. Single * means all unsafe sysctls are allowed.
   // Kubelet has to whitelist all allowed unsafe sysctls explicitly to avoid rejection.
   //
   // Examples:
   // e.g. "foo/*" allows "foo/bar", "foo/baz", etc.
   // e.g. "foo.*" allows "foo.bar", "foo.baz", etc.
   // +optional
   AllowedUnsafeSysctls []string `json:"allowedUnsafeSysctls,omitempty" protobuf:"bytes,19,rep,name=allowedUnsafeSysctls"`
   // forbiddenSysctls is a list of explicitly forbidden sysctls, defaults to none.
   // Each entry is either a plain sysctl name or ends in "*" in which case it is considered
   // as a prefix of forbidden sysctls. Single * means all sysctls are forbidden.
   //
   // Examples:
   // e.g. "foo/*" forbids "foo/bar", "foo/baz", etc.
   // e.g. "foo.*" forbids "foo.bar", "foo.baz", etc.
   // +optional
   ForbiddenSysctls []string `json:"forbiddenSysctls,omitempty" protobuf:"bytes,20,rep,name=forbiddenSysctls"`
   // AllowedProcMountTypes is a whitelist of allowed ProcMountTypes.
   // Empty or nil indicates that only the DefaultProcMountType may be used.
   // This requires the ProcMountType feature flag to be enabled.
   // +optional
   AllowedProcMountTypes []v1.ProcMountType `json:"allowedProcMountTypes,omitempty" protobuf:"bytes,21,opt,name=allowedProcMountTypes"`
   // runtimeClass is the strategy that will dictate the allowable RuntimeClasses for a pod.
   // If this field is omitted, the pod's runtimeClassName field is unrestricted.
   // Enforcement of this field depends on the RuntimeClass feature gate being enabled.
   // +optional
   RuntimeClass *RuntimeClassStrategyOptions `json:"runtimeClass,omitempty" protobuf:"bytes,24,opt,name=runtimeClass"`
}
```

创建：

```http
POST /apis/policy/v1beta1/podsecuritypolicies
```

添加配置：

```http
PATCH /apis/policy/v1beta1/podsecuritypolicies/{name}
```

修改：

```http
PUT /apis/policy/v1beta1/podsecuritypolicies/{name}
```

删除：

```http
DELETE /apis/policy/v1beta1/podsecuritypolicies/{name}
```

删除一堆：

```http
DELETE /apis/policy/v1beta1/podsecuritypolicies
```

读取：

```http
GET /apis/policy/v1beta1/podsecuritypolicies/{name}
```

读取列表：

```http
GET /apis/policy/v1beta1/podsecuritypolicies
```





















