# Kubernetes API学习(十七) - storage

```
$ kubectl api-resources --api-group=storage.k8s.io
NAME                SHORTNAMES   APIGROUP         NAMESPACED   KIND
csidrivers                       storage.k8s.io   false        CSIDriver
csinodes                         storage.k8s.io   false        CSINode
storageclasses      sc           storage.k8s.io   false        StorageClass
volumeattachments                storage.k8s.io   false        VolumeAttachment
```

三个 CSI 相关的资源，一个 StorageClass

源码在 API源码包的 storage 包下。



## StorageClass

```go
type StorageClass struct {
   metav1.TypeMeta `json:",inline"`
   // Standard object's metadata.
   // More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
   // +optional
   metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

   // 提供者
   Provisioner string `json:"provisioner" protobuf:"bytes,2,opt,name=provisioner"`

   // 应创建此存储类的卷的设置程序的参数。
   Parameters map[string]string `json:"parameters,omitempty" protobuf:"bytes,3,rep,name=parameters"`

   // 回收策略，有 Recycle，Delete，Retain
   ReclaimPolicy *v1.PersistentVolumeReclaimPolicy `json:"reclaimPolicy,omitempty" protobuf:"bytes,4,opt,name=reclaimPolicy,casttype=k8s.io/api/core/v1.PersistentVolumeReclaimPolicy"`

   // 这个存储类的动态配置的PersistentVolumes是用这些mountOptions创建的，例如。[“ro”，“soft”]。
   // 未验证-如果一个pv是无效的，那么pv的装载将失败。
   MountOptions []string `json:"mountOptions,omitempty" protobuf:"bytes,5,opt,name=mountOptions"`

   // AllowVolumeExpansion 显示存储类是否允许卷扩展
   // +optional
   AllowVolumeExpansion *bool `json:"allowVolumeExpansion,omitempty" protobuf:"varint,6,opt,name=allowVolumeExpansion"`

   // VolumeBindingMode指示应如何设置和绑定PersistentVolumeClaims。
   // 未设置时，使用VolumeBindingImmediate。
   // 只有启用VolumeScheduling功能的服务器才会使用此字段。
   VolumeBindingMode *VolumeBindingMode `json:"volumeBindingMode,omitempty" protobuf:"bytes,7,opt,name=volumeBindingMode"`

   // 限制可以动态配置卷的节点拓扑。
   // 每个卷插件都定义了自己支持的拓扑规范。
   // 空的拓扑SelectorTerm列表表示没有拓扑限制。
   // 只有启用VolumeScheduling功能的服务器才会使用此字段。
   AllowedTopologies []v1.TopologySelectorTerm `json:"allowedTopologies,omitempty" protobuf:"bytes,8,rep,name=allowedTopologies"`
}
```

创建：

```http
POST /apis/storage.k8s.io/v1/storageclasses
```

添加配置：

```http
PATCH /apis/storage.k8s.io/v1/storageclasses/{name}
```

修改：

```http
PUT /apis/storage.k8s.io/v1/storageclasses/{name}
```

删除：

```http
DELETE /apis/storage.k8s.io/v1/storageclasses/{name}
```

删除一堆：

```http
DELETE /apis/storage.k8s.io/v1/storageclasses
```

读取：

```http
GET /apis/storage.k8s.io/v1/storageclasses/{name}
```

读取列表：

```http
GET /apis/storage.k8s.io/v1/storageclasses
```



## VolumeAttachment

```go
type VolumeAttachment struct {
   metav1.TypeMeta `json:",inline"`

   // Standard object metadata.
   // More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
   // +optional
   metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

   // Specification of the desired attach/detach volume behavior.
   // Populated by the Kubernetes system.
   Spec VolumeAttachmentSpec `json:"spec" protobuf:"bytes,2,opt,name=spec"`

   // Status of the VolumeAttachment request.
   // Populated by the entity completing the attach or detach
   // operation, i.e. the external-attacher.
   // +optional
   Status VolumeAttachmentStatus `json:"status,omitempty" protobuf:"bytes,3,opt,name=status"`
}
```

VolumeAttachmentSpec

```go
type VolumeAttachmentSpec struct {
   // Attacher 指示必须处理此问题的卷驱动程序的名称
   // request. This is the name returned by GetPluginName().
   Attacher string `json:"attacher" protobuf:"bytes,1,opt,name=attacher"`

   // Source 代表应附加的卷。
   Source VolumeAttachmentSource `json:"source" protobuf:"bytes,2,opt,name=source"`

   // 卷应附加到的节点。
   NodeName string `json:"nodeName" protobuf:"bytes,3,opt,name=nodeName"`
}
```

VolumeAttachmentStatus

```go
type VolumeAttachmentStatus struct {
   // 指示卷已成功连接。
   // 此字段只能由完成附件的实体设置
   // operation, i.e. the external-attacher.
   Attached bool `json:"attached" protobuf:"varint,1,opt,name=attached"`

   // 成功附加后，该字段将填充必须由附加操作返回的任何信息，这些信息必须传递到后续的WaitForAttach或Mount调用中。
   // 此字段只能由完成附件的实体设置
   // operation, i.e. the external-attacher.
   // +optional
   AttachmentMetadata map[string]string `json:"attachmentMetadata,omitempty" protobuf:"bytes,2,rep,name=attachmentMetadata"`

   // 附加操作期间遇到的最后一个错误（如果有）。
   // 此字段只能由完成附件的实体设置
   // operation, i.e. the external-attacher.
   // +optional
   AttachError *VolumeError `json:"attachError,omitempty" protobuf:"bytes,3,opt,name=attachError,casttype=VolumeError"`

   // 分离操作期间遇到的最后一个错误（如果有）。
   // 此字段只能由完成附件的实体设置
   // operation, i.e. the external-attacher.
   // +optional
   DetachError *VolumeError `json:"detachError,omitempty" protobuf:"bytes,4,opt,name=detachError,casttype=VolumeError"`
}
```

创建：

```http
POST /apis/storage.k8s.io/v1/volumeattachments
```

添加配置：

```http
PATCH /apis/storage.k8s.io/v1/volumeattachments/{name}
```

修改：

```http
PUT /apis/storage.k8s.io/v1/volumeattachments/{name}
```

删除：

```http
DELETE /apis/storage.k8s.io/v1/volumeattachments/{name}
```

删除一堆：

```http
DELETE /apis/storage.k8s.io/v1/volumeattachments
```

读取：

```http
GET /apis/storage.k8s.io/v1/volumeattachments/{name}
```

读取列表：

```http
GET /apis/storage.k8s.io/v1/volumeattachments
```

添加状态：

```http
PATCH /apis/storage.k8s.io/v1/volumeattachments/{name}/status
```

读取状态：

```http
GET /apis/storage.k8s.io/v1/volumeattachments/{name}/status
```

修改状态：

```http
PUT /apis/storage.k8s.io/v1/volumeattachments/{name}/status
```



## CSINode

```go
type CSINode struct {
   metav1.TypeMeta `json:",inline"`

   // metadata.name must be the Kubernetes node name.
   metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

   // spec is the specification of CSINode
   Spec CSINodeSpec `json:"spec" protobuf:"bytes,2,opt,name=spec"`
}
```

CSINodeSpec:

```go
type CSINodeSpec struct {
   // drivers is a list of information of all CSI Drivers existing on a node.
   // If all drivers in the list are uninstalled, this can become empty.
   // +patchMergeKey=name
   // +patchStrategy=merge
   Drivers []CSINodeDriver `json:"drivers" patchStrategy:"merge" patchMergeKey:"name" protobuf:"bytes,1,rep,name=drivers"`
}
```

CSINodeDriver

```go
type CSINodeDriver struct {
   // 驱动器名字
   // This is the name of the CSI driver that this object refers to.
   // This MUST be the same name returned by the CSI GetPluginName() call for
   // that driver.
   Name string `json:"name" protobuf:"bytes,1,opt,name=name"`

   // 节点ID
   // nodeID of the node from the driver point of view.
   // This field enables Kubernetes to communicate with storage systems that do
   // not share the same nomenclature for nodes. For example, Kubernetes may
   // refer to a given node as "node1", but the storage system may refer to
   // the same node as "nodeA". When Kubernetes issues a command to the storage
   // system to attach a volume to a specific node, it can use this field to
   // refer to the node name using the ID that the storage system will
   // understand, e.g. "nodeA" instead of "node1". This field is required.
   NodeID string `json:"nodeID" protobuf:"bytes,2,opt,name=nodeID"`

   // topologyKeys is the list of keys supported by the driver.
   // When a driver is initialized on a cluster, it provides a set of topology
   // keys that it understands (e.g. "company.com/zone", "company.com/region").
   // When a driver is initialized on a node, it provides the same topology keys
   // along with values. Kubelet will expose these topology keys as labels
   // on its own node object.
   // When Kubernetes does topology aware provisioning, it can use this list to
   // determine which labels it should retrieve from the node object and pass
   // back to the driver.
   // It is possible for different nodes to use different topology keys.
   // This can be empty if driver does not support topology.
   // +optional
   TopologyKeys []string `json:"topologyKeys" protobuf:"bytes,3,rep,name=topologyKeys"`

   // allocatable represents the volume resources of a node that are available for scheduling.
   // This field is beta.
   // +optional
   Allocatable *VolumeNodeResources `json:"allocatable,omitempty" protobuf:"bytes,4,opt,name=allocatable"`
}
```

创建：

```http
POST /apis/storage.k8s.io/v1/csinodes
```

添加配置：

```http
PATCH /apis/storage.k8s.io/v1/csinodes/{name}
```

修改：

```http
PUT /apis/storage.k8s.io/v1/csinodes/{name}
```

删除：

```http
DELETE /apis/storage.k8s.io/v1/csinodes/{name}
```

删除一堆：

```http
DELETE /apis/storage.k8s.io/v1/csinodes
```

读取：

```http
GET /apis/storage.k8s.io/v1/csinodes/{name}
```

读取列表：

```http
GET /apis/storage.k8s.io/v1/csinodes
```



## CSIDriver

```go
type CSIDriver struct {
   metav1.TypeMeta `json:",inline"`

   // Standard object metadata.
   // metadata.Name indicates the name of the CSI driver that this object
   // refers to; it MUST be the same name returned by the CSI GetPluginName()
   // call for that driver.
   // The driver name must be 63 characters or less, beginning and ending with
   // an alphanumeric character ([a-z0-9A-Z]) with dashes (-), dots (.), and
   // alphanumerics between.
   // More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
   metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

   // Specification of the CSI Driver.
   Spec CSIDriverSpec `json:"spec" protobuf:"bytes,2,opt,name=spec"`
}
```

CSIDriverSpec

```go
type CSIDriverSpec struct {
   // attachRequired indicates this CSI volume driver requires an attach
   // operation (because it implements the CSI ControllerPublishVolume()
   // method), and that the Kubernetes attach detach controller should call
   // the attach volume interface which checks the volumeattachment status
   // and waits until the volume is attached before proceeding to mounting.
   // The CSI external-attacher coordinates with CSI volume driver and updates
   // the volumeattachment status when the attach operation is complete.
   // If the CSIDriverRegistry feature gate is enabled and the value is
   // specified to false, the attach operation will be skipped.
   // Otherwise the attach operation will be called.
   // +optional
   AttachRequired *bool `json:"attachRequired,omitempty" protobuf:"varint,1,opt,name=attachRequired"`

   // If set to true, podInfoOnMount indicates this CSI volume driver
   // requires additional pod information (like podName, podUID, etc.) during
   // mount operations.
   // If set to false, pod information will not be passed on mount.
   // Default is false.
   // The CSI driver specifies podInfoOnMount as part of driver deployment.
   // If true, Kubelet will pass pod information as VolumeContext in the CSI
   // NodePublishVolume() calls.
   // The CSI driver is responsible for parsing and validating the information
   // passed in as VolumeContext.
   // The following VolumeConext will be passed if podInfoOnMount is set to true.
   // This list might grow, but the prefix will be used.
   // "csi.storage.k8s.io/pod.name": pod.Name
   // "csi.storage.k8s.io/pod.namespace": pod.Namespace
   // "csi.storage.k8s.io/pod.uid": string(pod.UID)
   // "csi.storage.k8s.io/ephemeral": "true" iff the volume is an ephemeral inline volume
   //                                 defined by a CSIVolumeSource, otherwise "false"
   //
   // "csi.storage.k8s.io/ephemeral" is a new feature in Kubernetes 1.16. It is only
   // required for drivers which support both the "Persistent" and "Ephemeral" VolumeLifecycleMode.
   // Other drivers can leave pod info disabled and/or ignore this field.
   // As Kubernetes 1.15 doesn't support this field, drivers can only support one mode when
   // deployed on such a cluster and the deployment determines which mode that is, for example
   // via a command line parameter of the driver.
   // +optional
   PodInfoOnMount *bool `json:"podInfoOnMount,omitempty" protobuf:"bytes,2,opt,name=podInfoOnMount"`

   // VolumeLifecycleModes defines what kind of volumes this CSI volume driver supports.
   // The default if the list is empty is "Persistent", which is the usage
   // defined by the CSI specification and implemented in Kubernetes via the usual
   // PV/PVC mechanism.
   // The other mode is "Ephemeral". In this mode, volumes are defined inline
   // inside the pod spec with CSIVolumeSource and their lifecycle is tied to
   // the lifecycle of that pod. A driver has to be aware of this
   // because it is only going to get a NodePublishVolume call for such a volume.
   // For more information about implementing this mode, see
   // https://kubernetes-csi.github.io/docs/ephemeral-local-volumes.html
   // A driver can support one or more of these modes and
   // more modes may be added in the future.
   // +optional
   VolumeLifecycleModes []VolumeLifecycleMode `json:"volumeLifecycleModes,omitempty" protobuf:"bytes,3,opt,name=volumeLifecycleModes"`
}
```

读取：

```http
POST /apis/storage.k8s.io/v1beta1/csidrivers
```

添加配置：

```http
PATCH /apis/storage.k8s.io/v1beta1/csidrivers/{name}
```

修改：

```http
PUT /apis/storage.k8s.io/v1beta1/csidrivers/{name}
```

删除：

```http
DELETE /apis/storage.k8s.io/v1beta1/csidrivers/{name}
```

删除一堆：

```http
DELETE /apis/storage.k8s.io/v1beta1/csidrivers
```

读取：

```http
GET /apis/storage.k8s.io/v1beta1/csidrivers/{name}
```

读取列表：

```http
GET /apis/storage.k8s.io/v1beta1/csidrivers
```











