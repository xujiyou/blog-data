# Istio 错误记录

istio-telemetry-748d4fcf76-wzxpw 处于 CrashLoopBackOff 状态，一直重启，原因是 mixer 容器报错：

```
2020-02-07T03:43:53.444133Z     info    mcp     (re)trying to establish new MCP sink stream
2020-02-07T03:43:53.444269Z     error   mcp     Failed to create a new MCP sink stream: rpc error: code = Unavailable desc = all SubConns are in TransientFailure, latest connection error: connection error: desc = "transport: Error while dialing dial tcp 10.43.79.86:9901: connect: connection timed out"
```

解决方法：

添加 crd：

```yaml
kind: CustomResourceDefinition
apiVersion: apiextensions.k8s.io/v1beta1
metadata:
  name: authorizationpolicies.rbac.istio.io
  labels:
    app: istio-pilot
    istio: rbac
    heritage: Tiller
    release: istio
spec:
  group: rbac.istio.io
  names:
    kind: AuthorizationPolicy
    plural: authorizationpolicies
    singular: authorizationpolicy
    categories:
      - istio-io
      - rbac-istio-io
  scope: Namespaced
  version: v1alpha1
```

使用 kubectl apply -f 添加后，再等这个pod重启一次就可以了！！！