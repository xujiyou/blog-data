# Istio crd （二）- authentication

关于认证策略的 CRD。

官方教程：https://istio.io/zh/docs/tasks/security/authentication/authn-policy/

`authentication.istio.io` 一共有俩 CRD。分别是 `MeshPolicy` 和 `Policy` :

```bash
$ kubectl api-resources --api-group=authentication.istio.io
NAME           SHORTNAMES   APIGROUP                  NAMESPACED   KIND
meshpolicies                authentication.istio.io   false        MeshPolicy
policies                    authentication.istio.io   true         Policy
```



## MeshPolicy

每个集群中，都有一个 `MeshPolicy`，名为 default，这个资源对象是没有命名空间限制的。

MeshPolicy 用于设置网格范围的认证策略。

我的集群中默认的 MeshPolicy：

```yaml
$ kubectl get MeshPolicy default -o yaml
apiVersion: authentication.istio.io/v1alpha1
kind: MeshPolicy
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"authentication.istio.io/v1alpha1","kind":"MeshPolicy","metadata":{"annotations":{},"labels":{"app":"security","chart":"security","heritage":"Tiller","release":"cluster-istio"},"
name":"default"},"spec":{"peers":[{"mtls":{"mode":"PERMISSIVE"}}]}}
  creationTimestamp: "2020-04-06T04:49:33Z"
  generation: 1
  labels:
    app: security
    chart: security
    heritage: Tiller
    release: cluster-istio
  name: default
  resourceVersion: "11490"
  selfLink: /apis/authentication.istio.io/v1alpha1/meshpolicies/default
  uid: 4f1e7428-9578-4fa5-9ebf-1da7c398853b
spec:
  peers:
  - mtls:
      mode: PERMISSIVE
```

`PERMISSIVE` 的意思是放纵的，宽容的。可以看出默认是不启用全局双向 TLS 认证策略的。

如果像开启全局双向 TLS 认证策略，可以这样：

```bash
$ kubectl edit MeshPolicy default
```

修改为：

```yaml
apiVersion: authentication.istio.io/v1alpha1
kind: MeshPolicy
metadata:
  name: default
spec:
  peers:
  - mtls: {}
```

修改完成后，网格内所有的访问，如不指定特殊规则，默认都会启用双向 TLS 认证了。

之后，如果想要访问服务，就需要为每一个 `DestinationRule` 都配置上 `trafficPolicy` 。如下：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: hello
  namespace: xujiyou-test
spec:
  host: hello-service
  subsets:
    - name: default
      labels:
        app: hello
    - name: v1
      labels:
        version: v1
    - name: v2
      labels:
        version: v2
    - name: v3
      labels:
        version: v3
  trafficPolicy:
    tls:
      mode: ISTIO_MUTUAL
```





## Policy

在  [Istio服务保护.md](../实践/Istio服务保护.md) 的认证的那一节中，已经体验到 Policy 的使用了。

Policy 是命名空间级别的认证策略，如果不设置 targets，则默认是匹配指定的命名空间内的所有 service 。

查看 Policy 的定义：

```go
type Policy struct {
	// List rules to select workloads that the policy should be applied on.
	// If empty, policy will be used on all workloads in the same namespace.
	Targets []*TargetSelector `protobuf:"bytes,1,rep,name=targets,proto3" json:"targets,omitempty"`
	// List of authentication methods that can be used for peer authentication.
	// They will be evaluated in order; the first validate one will be used to
	// set peer identity (source.user) and other peer attributes. If none of
	// these methods pass, request will be rejected with authentication failed error (401).
	// Leave the list empty if peer authentication is not required
	Peers []*PeerAuthenticationMethod `protobuf:"bytes,2,rep,name=peers,proto3" json:"peers,omitempty"`
	// Set this flag to true to accept request (for peer authentication perspective),
	// even when none of the peer authentication methods defined above satisfied.
	// Typically, this is used to delay the rejection decision to next layer (e.g
	// authorization).
	// This flag is ignored if no authentication defined for peer (peers field is empty).
	PeerIsOptional bool `protobuf:"varint,3,opt,name=peer_is_optional,json=peerIsOptional,proto3" json:"peer_is_optional,omitempty"`
	// List of authentication methods that can be used for origin authentication.
	// Similar to peers, these will be evaluated in order; the first validate one
	// will be used to set origin identity and attributes (i.e request.auth.user,
	// request.auth.issuer etc). If none of these methods pass, request will be
	// rejected with authentication failed error (401).
	// A method may be skipped, depends on its trigger rule. If all of these methods
	// are skipped, origin authentication will be ignored, as if it is not defined.
	// Leave the list empty if origin authentication is not required.
	Origins []*OriginAuthenticationMethod `protobuf:"bytes,4,rep,name=origins,proto3" json:"origins,omitempty"`
	// Set this flag to true to accept request (for origin authentication perspective),
	// even when none of the origin authentication methods defined above satisfied.
	// Typically, this is used to delay the rejection decision to next layer (e.g
	// authorization).
	// This flag is ignored if no authentication defined for origin (origins field is empty).
	OriginIsOptional bool `protobuf:"varint,5,opt,name=origin_is_optional,json=originIsOptional,proto3" json:"origin_is_optional,omitempty"`
	// Define whether peer or origin identity should be use for principal. Default
	// value is USE_PEER.
	// If peer (or origin) identity is not available, either because of peer/origin
	// authentication is not defined, or failed, principal will be left unset.
	// In other words, binding rule does not affect the decision to accept or
	// reject request.
	PrincipalBinding     PrincipalBinding `protobuf:"varint,6,opt,name=principal_binding,json=principalBinding,proto3,enum=istio.authentication.v1alpha1.PrincipalBinding" json:"principal_binding,omitempty"`
	XXX_NoUnkeyedLiteral struct{}         `json:"-"`
	XXX_unrecognized     []byte           `json:"-"`
	XXX_sizecache        int32            `json:"-"`
}
```



