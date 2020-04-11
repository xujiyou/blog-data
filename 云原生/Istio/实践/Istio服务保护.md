# Istio 服务保护

Istio的安全功能十分强大，安全场景包括对网关的加密、服务间的访问控制、认证和授权。网关加密由Ingress Gateway实现，访问控制依赖于Mixer，认证和授权主要由Citadel、Envoy实现。



## 网关加密

HTTPS 能最大化地保证信息传输的安全，是当前互联网推荐的通信方式。Istio 为Gateway 提供了HTTPS加密支持。

一般的 We b 应用都采用单向认证，即仅客户端验证服务端证书，无须在通信层做用户身份验证，而是在应用逻辑层保证用户的合法登入。 

官方教程：https://istio.io/zh/docs/tasks/traffic-management/ingress/secure-ingress-mount/

SDS方式：https://istio.io/zh/docs/tasks/traffic-management/ingress/secure-ingress-sds/

#### 文件挂载方式

1. 创建一个 Kubernetes secret 以保存服务器的证书和私钥。使用 `kubectl` 在命名空间 `istio-system` 下创建 secret `istio-ingressgateway-certs`。Istio 网关将会自动加载该 secret。

   ```bash
   $ kubectl create -n istio-system secret tls istio-ingressgateway-certs --key kube-apiserver/apiserver-key.pem --cert kube-apiserver/apiserver.pem 
   ```

   检查 secret 是否挂载好了：

   ```bash
   $ kubectl exec -it -n istio-system $(kubectl -n istio-system get pods -l istio=ingressgateway -o jsonpath='{.items[0].metadata.name}') -- ls -al /etc/istio/ingressgateway-certs
   ```

2. 定义 Gateway 如下：



```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  namespace: xujiyou-test
  name: hello-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
    - port:
        number: 443
        name: https
        protocol: HTTPS
      hosts:
        - "fueltank-1.cloud.bbdops.com"
      tls:
        mode: SIMPLE
        serverCertificate: /etc/istio/ingressgateway-certs/tls.crt
        privateKey: /etc/istio/ingressgateway-certs/tls.key
```



1. 访问如下：

   ```bash
    $ curl -v -HHost:fueltank-1.cloud.bbdops.com --cacert /home/admin/k8s-cluster/cert/ca.pem https://fueltank-1.cloud.bbdops.com:31390/api/hello
   ```




## 认证

定义 hello-policy 文件如下：

```yaml
apiVersion: authentication.istio.io/v1alpha1
kind: Policy
metadata:
  name: hello-policy
  namespace: xujiyou-test
spec:
  peers:
    - mtls: {}
  targets:
    - name: hello-service
```

创建：

```bash
$  kubectl apply -f hello-policy.yaml
```

创建完成后，再使用上边的 curl 访问就会出错，因为 从 ingress-gateway 到 hello-service 这一块并没有进行认证。

修改 hello-destination-rule.yaml 如下，使用 SDS 自动添加认证：

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

`ISTIO_MUTUAL` 的意思就是通过 SDS 自动添加认证！

提交修改后，通过上边的 curl 通过 ingress-gateway 又可以访问了。但是直接通过 Pod 和 Service 是无法访问的。

​	`mtls` 中的 m 就是 `MUTUAL` 的意思，这个单词是双向的意思，`mtls` 就是双向认证的意思！！！



#### jwt 认证

这里有一个巨坑！！！ kubernetes 的 service 的端口名要指定！！！如下，要指定 `name: http`：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-service
  namespace: xujiyou-test
  labels:
    app: hello
spec:
  ports:
    - port: 8080
      name: http
  selector:
    app: hello
  type: NodePort
```

修改 hello-policy 如下：

```yaml
apiVersion: authentication.istio.io/v1alpha1
kind: Policy
metadata:
  name: hello-policy
  namespace: xujiyou-test
spec:
  origins:
    - jwt:
        issuer: "testing@secure.istio.io"
        jwksUri: "http://fueltank-1:81/jwks.json"
  principalBinding: USE_ORIGIN
  peers:
    - mtls: {}
  targets:
    - name: hello-service
```

创建：

```bash
$ kubectl apply -f hello-policy.yaml
```

创建之后，再访问：

```bash
$ curl -v -HHost:fueltank-1.cloud.bbdops.com --cacert /home/admin/k8s-cluster/cert/ca.pem https://fueltank-1.cloud.bbdops.com:31390/api/hello?version=v2
```

会发现返回 401，怎么办那，可以这样：

```bash
$ TOKEN=$(curl https://raw.githubusercontent.com/istio/istio/release-1.5/security/tools/jwt/samples/demo.jwt -s)
$ curl -v -HHost:fueltank-1.cloud.bbdops.com --cacert /home/admin/k8s-cluster/cert/ca.pem https://fueltank-1.cloud.bbdops.com:31390/api/hello?version=v2 --header "Authorization: Bearer $TOKEN"
```



## 授权

Istio 1.4 版本之后，开始使用 `AuthorizationPolicy` ，不再推荐使用 `rbac.istio.io` 中的 CRD。

等到 Istio 1.6 版本，`rbac.istio.io` 将被删除！！！

下来来看看怎么使用。

**注意，Kubernetes 的原生 Service 要对端口号进行命名，切记切记**

下面先来看 hello-authorization-policy.yaml :

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: hello-authorization-policy
  namespace: xujiyou-test
spec:
  selector:
    app: hello
  rules:
    - from:
        - source:
            principals:
              - cluster.local/ns/istio-system/sa/istio-ingressgateway-service-account
      to:
        - operation:
            methods:
              - GET
              - POST
      when:
        - key: request.headers[User-Agent]
          values:
            - "curl/*"
```

`principals` 中使用的是 Pod 中的 ServiceAccount ！！！

这个文件的意思是 ：hello 服务只允许名为 `istio-ingressgateway-service-account` 的 ServiceAccount 对本服务进行 GET 请求，并且在请求的 headers 中要带有 `User-Agent: curl/*` ，即只允许 curl 访问。

`when` 中完整的 key 定义请查看：https://istio.io/docs/reference/config/security/conditions/ ，其 value 支持通配符。

只对一个服务进行了授权之后，也会自动为所在命名空间下的其他服务添加禁止访问的权限。

因为 hello 服务还访问了 other 服务，所以这里也要为 other 服务添加授权策略。

编写 other-authorization-policy.yaml :

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: other-authorization-policy
  namespace: xujiyou-test
spec:
  selector:
    app: other
  rules:
    - from:
        - source:
            namespaces:
              - xujiyou-test
            principals:
              - "*"
              - cluster.local/ns/xujiyou-test/sa/default
      to:
        - operation:
            methods:
              - GET
```

注意看，这里的 source 不仅可以定义 ServiceAccount ，还支持定义命名空间和通配符。

在 Istio 1.5 版本之后，Istio 将不会再有黑名单白名单之类的访问控制，而是同意使用 `AuthorizationPolicy` 进行访问控制！！！





