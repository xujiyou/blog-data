# Kubernetes 学习（八）- networking

查看对象列表：

```bash
$ kubectl api-resources --api-group=networking.k8s.io
NAME              SHORTNAMES   APIGROUP            NAMESPACED   KIND
ingresses         ing          networking.k8s.io   true         Ingress
networkpolicies   netpol       networking.k8s.io   true         NetworkPolicy
```



## Ingress

对象结构：

```go
type Ingress struct {
   metav1.TypeMeta `json:",inline"`
   // Standard object's metadata.
   // More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
   // +optional
   metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

   // Spec is the desired state of the Ingress.
   // More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
   // +optional
   Spec IngressSpec `json:"spec,omitempty" protobuf:"bytes,2,opt,name=spec"`

   // Status is the current state of the Ingress.
   // More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
   // +optional
   Status IngressStatus `json:"status,omitempty" protobuf:"bytes,3,opt,name=status"`
}
```

IngressSpec:

```go
type IngressSpec struct {
   // ingress 的后端，指定 server 的名字和端口
   Backend *IngressBackend `json:"backend,omitempty" protobuf:"bytes,1,opt,name=backend"`

   // 使用的证书及私钥等
   TLS []IngressTLS `json:"tls,omitempty" protobuf:"bytes,2,rep,name=tls"`

   // 规则
   Rules []IngressRule `json:"rules,omitempty" protobuf:"bytes,3,rep,name=rules"`
   // TODO: Add the ability to specify load-balancer IP through claims
}
```

IngressStatus

```go
type IngressStatus struct {
   // LoadBalancer contains the current status of the load-balancer.
   // +optional
   LoadBalancer v1.LoadBalancerStatus `json:"loadBalancer,omitempty" protobuf:"bytes,1,opt,name=loadBalancer"`
}
```

创建：

```http
POST /apis/networking.k8s.io/v1beta1/namespaces/{namespace}/ingresses
```

添加配置：

```http
PATCH /apis/networking.k8s.io/v1beta1/namespaces/{namespace}/ingresses/{name}
```

修改：

```http
PUT /apis/networking.k8s.io/v1beta1/namespaces/{namespace}/ingresses/{name}
```

删除：

```http
DELETE /apis/networking.k8s.io/v1beta1/namespaces/{namespace}/ingresses/{name}
```

删除一堆：

```http
DELETE /apis/networking.k8s.io/v1beta1/namespaces/{namespace}/ingresses
```

读取：

```http
GET /apis/networking.k8s.io/v1beta1/namespaces/{namespace}/ingresses/{name}
```

读取列表：

```http
GET /apis/networking.k8s.io/v1beta1/namespaces/{namespace}/ingresses
```

获取全部命名空间的：

```http
GET /apis/networking.k8s.io/v1beta1/ingresses
```

添加状态：

```http
PATCH /apis/networking.k8s.io/v1beta1/namespaces/{namespace}/ingresses/{name}/status
```

读取状态：

```http
GET /apis/networking.k8s.io/v1beta1/namespaces/{namespace}/ingresses/{name}/status
```

替换状态：

```http
PUT /apis/networking.k8s.io/v1beta1/namespaces/{namespace}/ingresses/{name}/status
```



## NetworkPolicy

详细介绍请看：https://kubernetes.io/zh/docs/concepts/services-networking/network-policies/

网络策略（NetworkPolicy）是一种关于pod间及pod与其他网络端点间所允许的通信规则的规范。

`NetworkPolicy` 资源使用标签选择pod，并定义选定pod所允许的通信规则。

网络策略通过网络插件来实现，所以用户必须使用支持 `NetworkPolicy` 的网络解决方案 - 简单地创建资源对象，而没有控制器来使它生效的话，是没有任何作用的。Calico 有这种控制器的实现！！！

对象结构：

```go
type NetworkPolicy struct {
	metav1.TypeMeta `json:",inline"`
	// Standard object's metadata.
	// More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata
	// +optional
	metav1.ObjectMeta `json:"metadata,omitempty" protobuf:"bytes,1,opt,name=metadata"`

	// Specification of the desired behavior for this NetworkPolicy.
	// +optional
	Spec NetworkPolicySpec `json:"spec,omitempty" protobuf:"bytes,2,opt,name=spec"`
}
```

NetworkPolicySpec:

```go
type NetworkPolicySpec struct {
	// Selects the pods to which this NetworkPolicy object applies. The array of
	// ingress rules is applied to any pods selected by this field. Multiple network
	// policies can select the same set of pods. In this case, the ingress rules for
	// each are combined additively. This field is NOT optional and follows standard
	// label selector semantics. An empty podSelector matches all pods in this
	// namespace.
	PodSelector metav1.LabelSelector `json:"podSelector" protobuf:"bytes,1,opt,name=podSelector"`

	// List of ingress rules to be applied to the selected pods. Traffic is allowed to
	// a pod if there are no NetworkPolicies selecting the pod
	// (and cluster policy otherwise allows the traffic), OR if the traffic source is
	// the pod's local node, OR if the traffic matches at least one ingress rule
	// across all of the NetworkPolicy objects whose podSelector matches the pod. If
	// this field is empty then this NetworkPolicy does not allow any traffic (and serves
	// solely to ensure that the pods it selects are isolated by default)
	// +optional
	Ingress []NetworkPolicyIngressRule `json:"ingress,omitempty" protobuf:"bytes,2,rep,name=ingress"`

	// List of egress rules to be applied to the selected pods. Outgoing traffic is
	// allowed if there are no NetworkPolicies selecting the pod (and cluster policy
	// otherwise allows the traffic), OR if the traffic matches at least one egress rule
	// across all of the NetworkPolicy objects whose podSelector matches the pod. If
	// this field is empty then this NetworkPolicy limits all outgoing traffic (and serves
	// solely to ensure that the pods it selects are isolated by default).
	// This field is beta-level in 1.8
	// +optional
	Egress []NetworkPolicyEgressRule `json:"egress,omitempty" protobuf:"bytes,3,rep,name=egress"`

	// List of rule types that the NetworkPolicy relates to.
	// Valid options are "Ingress", "Egress", or "Ingress,Egress".
	// If this field is not specified, it will default based on the existence of Ingress or Egress rules;
	// policies that contain an Egress section are assumed to affect Egress, and all policies
	// (whether or not they contain an Ingress section) are assumed to affect Ingress.
	// If you want to write an egress-only policy, you must explicitly specify policyTypes [ "Egress" ].
	// Likewise, if you want to write a policy that specifies that no egress is allowed,
	// you must specify a policyTypes value that include "Egress" (since such a policy would not include
	// an Egress section and would otherwise default to just [ "Ingress" ]).
	// This field is beta-level in 1.8
	// +optional
	PolicyTypes []PolicyType `json:"policyTypes,omitempty" protobuf:"bytes,4,rep,name=policyTypes,casttype=PolicyType"`
}
```

下面是一个示例：

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978
```

创建：

```http
POST /apis/networking.k8s.io/v1/namespaces/{namespace}/networkpolicies
```

添加配置：

```http
PATCH /apis/networking.k8s.io/v1/namespaces/{namespace}/networkpolicies/{name}
```

修改：

```http
PUT /apis/networking.k8s.io/v1/namespaces/{namespace}/networkpolicies/{name}
```

删除：

```http
DELETE /apis/networking.k8s.io/v1/namespaces/{namespace}/networkpolicies/{name}
```

删除一堆：

```http
DELETE /apis/networking.k8s.io/v1/namespaces/{namespace}/networkpolicies
```

读取：

```http
GET /apis/networking.k8s.io/v1/namespaces/{namespace}/networkpolicies/{name}
```

读取列表：

```http
GET /apis/networking.k8s.io/v1/namespaces/{namespace}/networkpolicies
```

读取全部命名空间的列表：

```http
GET /apis/networking.k8s.io/v1/networkpolicies
```





























