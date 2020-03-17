# Kubernetes API 学习（七）- APIService

查看对象：

```bash
$ kubectl api-resources --api-group=apiregistration.k8s.io
NAME          SHORTNAMES   APIGROUP                 NAMESPACED   KIND
apiservices                apiregistration.k8s.io   false        APIService
```

APIService是用来表示一个特定的`GroupVersion`的中的server，它的结构定义位于代码`staging/src/k8s.io/kube-aggregator/pkg/apis/apiregistration/types.go`中。

```go
type APIService struct {
   metav1.TypeMeta
   metav1.ObjectMeta

   // Spec contains information for locating and communicating with a server
   Spec APIServiceSpec
   // Status contains derived information about an API server
   Status APIServiceStatus
}
```

APIServiceSpec:

```go
type APIServiceSpec struct {
   // service 引用
   Service *ServiceReference
   // 组名
   Group string
   // 版本名
   Version string

   // 不安全，跳过验证
   InsecureSkipTLSVerify bool
   // CABundle is a PEM encoded CA bundle which will be used to validate an API server's serving certificate.
   // If unspecified, system trust roots on the apiserver are used.
   // +optional
   CABundle []byte

   // GroupPriorityMinimum is the priority this group should have at least. Higher priority means that the group is preferred by clients over lower priority ones.
   // Note that other versions of this group might specify even higher GroupPriorityMininum values such that the whole group gets a higher priority.
   // The primary sort is based on GroupPriorityMinimum, ordered highest number to lowest (20 before 10).
   // The secondary sort is based on the alphabetical comparison of the name of the object.  (v1.bar before v1.foo)
   // We'd recommend something like: *.k8s.io (except extensions) at 18000 and
   // PaaSes (OpenShift, Deis) are recommended to be in the 2000s
   GroupPriorityMinimum int32

   // VersionPriority controls the ordering of this API version inside of its group.  Must be greater than zero.
   // The primary sort is based on VersionPriority, ordered highest to lowest (20 before 10).
   // Since it's inside of a group, the number can be small, probably in the 10s.
   // In case of equal version priorities, the version string will be used to compute the order inside a group.
   // If the version string is "kube-like", it will sort above non "kube-like" version strings, which are ordered
   // lexicographically. "Kube-like" versions start with a "v", then are followed by a number (the major version),
   // then optionally the string "alpha" or "beta" and another number (the minor version). These are sorted first
   // by GA > beta > alpha (where GA is a version with no suffix such as beta or alpha), and then by comparing major
   // version, then minor version. An example sorted list of versions:
   // v10, v2, v1, v11beta2, v10beta3, v3beta1, v12alpha1, v11alpha2, foo1, foo10.
   VersionPriority int32
}
```

APIServiceStatus:

```go
type APIServiceStatus struct {
   // 状态列表
   Conditions []APIServiceCondition
}
```

创建：

```http
POST /apis/apiregistration.k8s.io/v1/apiservices
```

修改：

```http
PATCH /apis/apiregistration.k8s.io/v1/apiservices/{name}
```

替换：

```http
PUT /apis/apiregistration.k8s.io/v1/apiservices/{name}
```

删除：

```http
DELETE /apis/apiregistration.k8s.io/v1/apiservices/{name}
```

删除一堆：

```http
DELETE /apis/apiregistration.k8s.io/v1/apiservices
```

读取：

```http
GET /apis/apiregistration.k8s.io/v1/apiservices/{name}
```

读取列表：

```http
GET /apis/apiregistration.k8s.io/v1/apiservices
```

添加状态：

```http
PATCH /apis/apiregistration.k8s.io/v1/apiservices/{name}/status
```

读取状态：

```http
GET /apis/apiregistration.k8s.io/v1/apiservices/{name}/status
```

修改状态：

```http
PUT /apis/apiregistration.k8s.io/v1/apiservices/{name}/status
```





```bash
$ kubectl get apiservice
NAME                                   SERVICE   AVAILABLE   AGE
v1.                                    Local     True        4d6h
v1.admissionregistration.k8s.io        Local     True        4d6h
v1.apiextensions.k8s.io                Local     True        4d6h
v1.apps                                Local     True        4d6h
v1.authentication.k8s.io               Local     True        4d6h
v1.authorization.k8s.io                Local     True        4d6h
v1.autoscaling                         Local     True        4d6h
v1.batch                               Local     True        4d6h
v1.coordination.k8s.io                 Local     True        4d6h
v1.networking.k8s.io                   Local     True        4d6h
v1.rbac.authorization.k8s.io           Local     True        4d6h
v1.scheduling.k8s.io                   Local     True        4d6h
v1.storage.k8s.io                      Local     True        4d6h
v1alpha1.snapshot.storage.k8s.io       Local     True        24h
v1beta1.admissionregistration.k8s.io   Local     True        4d6h
v1beta1.apiextensions.k8s.io           Local     True        4d6h
v1beta1.authentication.k8s.io          Local     True        4d6h
v1beta1.authorization.k8s.io           Local     True        4d6h
v1beta1.batch                          Local     True        4d6h
v1beta1.certificates.k8s.io            Local     True        4d6h
v1beta1.coordination.k8s.io            Local     True        4d6h
v1beta1.discovery.k8s.io               Local     True        4d6h
v1beta1.events.k8s.io                  Local     True        4d6h
v1beta1.extensions                     Local     True        4d6h
v1beta1.networking.k8s.io              Local     True        4d6h
v1beta1.node.k8s.io                    Local     True        4d6h
v1beta1.policy                         Local     True        4d6h
v1beta1.rbac.authorization.k8s.io      Local     True        4d6h
v1beta1.scheduling.k8s.io              Local     True        4d6h
v1beta1.storage.k8s.io                 Local     True        4d6h
v2beta1.autoscaling                    Local     True        4d6h
v2beta2.autoscaling                    Local     True        4d6h
```





```bash
$ kubectl api-versions 
admissionregistration.k8s.io/v1
admissionregistration.k8s.io/v1beta1
apiextensions.k8s.io/v1
apiextensions.k8s.io/v1beta1
apiregistration.k8s.io/v1
apiregistration.k8s.io/v1beta1
apps/v1
authentication.k8s.io/v1
authentication.k8s.io/v1beta1
authorization.k8s.io/v1
authorization.k8s.io/v1beta1
autoscaling/v1
autoscaling/v2beta1
autoscaling/v2beta2
batch/v1
batch/v1beta1
certificates.k8s.io/v1beta1
coordination.k8s.io/v1
coordination.k8s.io/v1beta1
discovery.k8s.io/v1beta1
events.k8s.io/v1beta1
extensions/v1beta1
networking.k8s.io/v1
networking.k8s.io/v1beta1
node.k8s.io/v1beta1
policy/v1beta1
rbac.authorization.k8s.io/v1
rbac.authorization.k8s.io/v1beta1
scheduling.k8s.io/v1
scheduling.k8s.io/v1beta1
snapshot.storage.k8s.io/v1alpha1
storage.k8s.io/v1
storage.k8s.io/v1beta1
v1
```

