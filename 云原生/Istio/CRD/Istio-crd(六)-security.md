# Istio crd （六） - security

`security.istio.io` 中只有一个 CRD，名为 `AuthorizationPolicy` ，意味授权策略。

```
$ kubectl api-resources --api-group=security.istio.io 
NAME                    SHORTNAMES   APIGROUP            NAMESPACED   KIND
authorizationpolicies                security.istio.io   true         AuthorizationPolicy
```

Istio 1.4引入了 [`AuthorizationPolicy`](https://istio.io/docs/reference/config/security/authorization-policy/)，这是对以前`v1alpha1`的基于角色的访问控制（RBAC）策略的重大更新。

Istio 1.6及更高版本将不再支持先前的配置资源`ClusterRbacConfig`，`ServiceRole`和 `ServiceRoleBinding`。

