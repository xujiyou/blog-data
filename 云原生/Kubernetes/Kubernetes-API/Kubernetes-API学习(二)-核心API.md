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


