# Kubernetes API 学习（五）- RDBC

通过以下命令来查看有哪些资源对象：

```bash
$ kubectl api-resources --api-group=rbac.authorization.k8s.io
NAME                 SHORTNAMES   APIGROUP                   NAMESPACED   KIND
clusterrolebindings               rbac.authorization.k8s.io  false       ClusterRoleBinding
clusterroles                      rbac.authorization.k8s.io  false        ClusterRole
rolebindings                      rbac.authorization.k8s.io  true         RoleBinding
roles                             rbac.authorization.k8s.io  true         Role
```

一共四个资源对象。

源码位于 API 源码中的 rdbc 包。



## Role

对象结构：

```go
type Role struct {
	metav1.TypeMeta `json:",inline"`
	// Standard object's metadata.
	// +optional
	metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

	// Rules holds all the PolicyRules for this Role
	// +optional
	Rules []PolicyRule `json:"rules" protobuf:"bytes,2,rep,name=rules"`
}
```

PolicyRule:

```go
type PolicyRule struct {
   // 允许有哪些动作，比如 get、list、creat 等。
   Verbs []string `json:"verbs" protobuf:"bytes,1,rep,name=verbs"`

   // 资源组,如果指定了多个API组，则将允许对任何API组中的枚举资源之一请求的任何操作。
   APIGroups []string `json:"apiGroups,omitempty" protobuf:"bytes,2,rep,name=apiGroups"`
   // 资源是此规则应用于的资源的列表。ResourceAll表示所有资源。比如：podsecuritypolicies
   // +optional
   Resources []string `json:"resources,omitempty" protobuf:"bytes,3,rep,name=resources"`
   // ResourceNames是规则应用于的可选名称白名单。空集意味着一切都是允许的。比如一个 podsecuritypolicies 实例。
   // +optional
   ResourceNames []string `json:"resourceNames,omitempty" protobuf:"bytes,4,rep,name=resourceNames"`

   // 允许访问的 api 地址列表
   // NonResourceURLs is a set of partial urls that a user should have access to.  *s are allowed, but only as the full, final step in the path
   // Since non-resource URLs are not namespaced, this field is only applicable for ClusterRoles referenced from a ClusterRoleBinding.
   // Rules can either apply to API resources (such as "pods" or "secrets") or non-resource URL paths (such as "/api"),  but not both.
   // +optional
   NonResourceURLs []string `json:"nonResourceURLs,omitempty" protobuf:"bytes,5,rep,name=nonResourceURLs"`
}
```

创建：

```http
POST /apis/rbac.authorization.k8s.io/v1/namespaces/{namespace}/roles
```

添加配置：

```http
PATCH /apis/rbac.authorization.k8s.io/v1/namespaces/{namespace}/roles/{name}
```

修改：

```http
PUT /apis/rbac.authorization.k8s.io/v1/namespaces/{namespace}/roles/{name}
```

删除：

```http
DELETE /apis/rbac.authorization.k8s.io/v1/namespaces/{namespace}/roles/{name}
```

删除一堆：

```http
DELETE /apis/rbac.authorization.k8s.io/v1/namespaces/{namespace}/roles
```

读取：

```http
GET /apis/rbac.authorization.k8s.io/v1/namespaces/{namespace}/roles/{name}
```

读取列表：

```http
GET /apis/rbac.authorization.k8s.io/v1/namespaces/{namespace}/roles
```

读取所有命名空间的：

```http
GET /apis/rbac.authorization.k8s.io/v1/roles
```



## RoleBinding

对象结构：

```go
type RoleBinding struct {
	metav1.TypeMeta `json:",inline"`
	// Standard object's metadata.
	// +optional
	metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

	// Subjects holds references to the objects the role applies to.
	// +optional
	Subjects []Subject `json:"subjects,omitempty" protobuf:"bytes,2,rep,name=subjects"`

	// RoleRef can reference a Role in the current namespace or a ClusterRole in the global namespace.
	// If the RoleRef cannot be resolved, the Authorizer must return an error.
	RoleRef RoleRef `json:"roleRef" protobuf:"bytes,3,opt,name=roleRef"`
}
```

Subject:

```go
type Subject struct {
   //"User", "Group", and "ServiceAccount".
   Kind string `json:"kind" protobuf:"bytes,1,opt,name=kind"`
   // APIGroup holds the API group of the referenced subject.
   // Defaults to "" for ServiceAccount subjects.
   // Defaults to "rbac.authorization.k8s.io" for User and Group subjects.
   // +optional
   APIGroup string `json:"apiGroup,omitempty" protobuf:"bytes,2,opt.name=apiGroup"`
   // 名字
   Name string `json:"name" protobuf:"bytes,3,opt,name=name"`
   // Namespace of the referenced object.  If the object kind is non-namespace, such as "User" or "Group", and this value is not empty
   // the Authorizer should report an error.
   // +optional
   Namespace string `json:"namespace,omitempty" protobuf:"bytes,4,opt,name=namespace"`
}
```

RoleRef：

```go
type RoleRef struct {
   // 为 “rbac.authorization.k8s.io”
   APIGroup string `json:"apiGroup" protobuf:"bytes,1,opt,name=apiGroup"`
   // 为 “Role”
   Kind string `json:"kind" protobuf:"bytes,2,opt,name=kind"`
   // Role 的名字
   Name string `json:"name" protobuf:"bytes,3,opt,name=name"`
}
```

创建：

```http
POST /apis/rbac.authorization.k8s.io/v1/namespaces/{namespace}/rolebindings
```

添加配置：

```http
PATCH /apis/rbac.authorization.k8s.io/v1/namespaces/{namespace}/rolebindings/{name}
```

替换：

```http
PUT /apis/rbac.authorization.k8s.io/v1/namespaces/{namespace}/rolebindings/{name}
```

删除：

```http
DELETE /apis/rbac.authorization.k8s.io/v1/namespaces/{namespace}/rolebindings/{name}
```

删除一堆：

```http
DELETE /apis/rbac.authorization.k8s.io/v1/namespaces/{namespace}/rolebindings
```

读取：

```http
GET /apis/rbac.authorization.k8s.io/v1/namespaces/{namespace}/rolebindings/{name}
```

读取列表：

```http
GET /apis/rbac.authorization.k8s.io/v1/namespaces/{namespace}/rolebindings
```

读取全部命名空间的：

```http
GET /apis/rbac.authorization.k8s.io/v1/rolebindings
```



## ClusterRole

对象结构：

```go
type ClusterRole struct {
   metav1.TypeMeta `json:",inline"`
   // Standard object's metadata.
   // +optional
   metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

   // Rules holds all the PolicyRules for this ClusterRole
   // +optional
   Rules []PolicyRule `json:"rules" protobuf:"bytes,2,rep,name=rules"`

   // AggregationRule is an optional field that describes how to build the Rules for this ClusterRole.
   // If AggregationRule is set, then the Rules are controller managed and direct changes to Rules will be
   // stomped by the controller.
   // +optional
   AggregationRule *AggregationRule `json:"aggregationRule,omitempty" protobuf:"bytes,3,opt,name=aggregationRule"`
}
```

AggregationRule:

```go
type AggregationRule struct {
   // 选中多个 ClusterRole
   ClusterRoleSelectors []metav1.LabelSelector `json:"clusterRoleSelectors,omitempty" protobuf:"bytes,1,rep,name=clusterRoleSelectors"`
}
```

创建：

```
POST /apis/rbac.authorization.k8s.io/v1/clusterroles
```

添加配置：

```
PATCH /apis/rbac.authorization.k8s.io/v1/clusterroles/{name}
```

修改：

```
PUT /apis/rbac.authorization.k8s.io/v1/clusterroles/{name}
```

删除：

```
DELETE /apis/rbac.authorization.k8s.io/v1/clusterroles/{name}
```

删除一堆：

```
DELETE /apis/rbac.authorization.k8s.io/v1/clusterroles
```

读取：

```
GET /apis/rbac.authorization.k8s.io/v1/clusterroles/{name}
```

读取列表：

```
GET /apis/rbac.authorization.k8s.io/v1/clusterroles
```



## ClusterRoleBinding

对象结构：

```go
type ClusterRoleBinding struct {
   metav1.TypeMeta `json:",inline"`
   // Standard object's metadata.
   // +optional
   metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

   // Subjects holds references to the objects the role applies to.
   // +optional
   Subjects []Subject `json:"subjects,omitempty" protobuf:"bytes,2,rep,name=subjects"`

   // RoleRef can only reference a ClusterRole in the global namespace.
   // If the RoleRef cannot be resolved, the Authorizer must return an error.
   RoleRef RoleRef `json:"roleRef" protobuf:"bytes,3,opt,name=roleRef"`
}
```

创建：

```http
POST /apis/rbac.authorization.k8s.io/v1/clusterrolebindings
```

增加配置：

```http
PATCH /apis/rbac.authorization.k8s.io/v1/clusterrolebindings/{name}
```

修改：

```http
PUT /apis/rbac.authorization.k8s.io/v1/clusterrolebindings/{name}
```

删除：

```http
DELETE /apis/rbac.authorization.k8s.io/v1/clusterrolebindings/{name}
```

删除一堆：

```http
DELETE /apis/rbac.authorization.k8s.io/v1/clusterrolebindings
```

读取：

```http
GET /apis/rbac.authorization.k8s.io/v1/clusterrolebindings/{name}
```

读取列表：

```http
GET /apis/rbac.authorization.k8s.io/v1/clusterrolebindings
```

















