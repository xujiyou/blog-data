# Istio 流量治理

Istio 的流量治理包括负载均衡、会话保持、故障注入、超时、重试、熔断、限流、服务隔离等。

## 负载均衡

修改 other 服务的 DestinationRule：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: other
  namespace: xujiyou-test
spec:
  host: other-service
  subsets:
    - name: v1
      labels:
        version: v1.0
    - name: v2
      labels:
        version: v2.0
  trafficPolicy:
    loadBalancer:
      simple: ROUND_ROBIN
```

ROUND_ROBIN 的作用是平均分配。另外，还有 RANDOM （随机），

怎么查看效果那，只能查看 istio-proxy 容器的日志了。



## 会话保持

会话保持是将来自同一客户端的请求始终映射到同一个后端实例中，让请求具有记忆性。会话保持 带来的好处是：如果在服务端的缓存中保存着客户端的请求结果，且同一个客户端始终访问同一个后端 实例，就可以一直从缓存中获取数据。Istio利用一致性哈希算法提供了会话保持功能，也属于负载均衡算 法，这种负载均衡策略只对HTTP连接有效。 

也是对 DestinationRule 进行修改：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: other
  namespace: xujiyou-test
spec:
  host: other-service
  subsets:
    - name: v1
      labels:
        version: v1.0
    - name: v2
      labels:
        version: v2.0
  trafficPolicy:
    loadBalancer:
      consistentHash:
        httpCookie:
          name: user
          ttl: 60s
```



## 故障注入

故障注入是一种软件测试方式，通过在代码中引入故障来发现系统软件中隐藏的Bug，并且通常与压 力测试一起用于验证软件的稳健性。目前，Istio故障注入功能支持延迟注入和中断注入。

#### 延迟注入

延迟属于时序故障，模仿增加的网络延迟或过载的上游服务。故障配置可以设置在特定条件下对请 求注入故障，也可以限制发生请求故障的百分比。

修改 VirtualService：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  namespace: xujiyou-test
  name: other
spec:
  hosts:
    - other-service
  http:
    - fault:
        delay:
          fixedDelay: 3s
          percent: 100
      route:
        - weight: 80
          destination:
            host: other-service
            subset: v1
            port:
              number: 8081
        - weight: 20
          destination:
            host: other-service
            subset: v2
            port:
              number: 8081
```

访问：

```bash
$ curl http://fueltank-1/api/hello?version=v1
```

这样每次访问都会延迟三秒了。

####中断注入

再次修改上边的 VirtualService：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  namespace: xujiyou-test
  name: other
spec:
  hosts:
    - other-service
  http:
    - fault:
        abort:
          httpStatus: 500
          percent: 100
      route:
        - weight: 80
          destination:
            host: other-service
            subset: v1
            port:
              number: 8081
        - weight: 20
          destination:
            host: other-service
            subset: v2
            port:
              number: 8081
```

再访问，就直接返回 500错误了：

```
$ curl http://fueltank-1/api/hello?version=v1
{"timestamp":"2020-04-08T05:57:57.517+0000","status":500,"error":"Internal Server Error","message":"status 500 reading OtherClient#other()","path":"/api/hello"}
```



## 超时

接着使用上边的延迟注入，下面来修改 hello 服务，设置 1 秒超时：

修改 hello 项目的 VirtualService：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  namespace: xujiyou-test
  name: hello
spec:
  hosts:
    - "*"
  gateways:
    - hello-gateway
  http:
    - match:
        - uri:
            prefix: /api/hello
          queryParams:
            version:
              exact: v1
      route:
        - destination:
            host: hello-service
            subset: v1
            port:
              number: 8080
    - match:
        - uri:
            prefix: /api/hello
          queryParams:
            version:
              exact: v2
      route:
        - destination:
            host: hello-service
            subset: v2
            port:
              number: 8080
    - match:
        - uri:
            prefix: /api/hello
          queryParams:
            version:
              exact: v3
      route:
        - destination:
            host: hello-service
            subset: v3
            port:
              number: 8080
    - route:
        - destination:
            host: hello-service
            subset: default
            port:
              number: 8080
      timeout: 1s
```

仅仅是在底部加了一个 timeout。下面来测试：

```bash
$ curl http://fueltank-1/api/hello
upstream request timeout[
```



## 重试

接上面的中断注入，然后在 hello 服务上配置遇到 500 错误 就重试三次：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  namespace: xujiyou-test
  name: hello
spec:
  hosts:
    - "*"
  gateways:
    - hello-gateway
  http:
    - match:
        - uri:
            prefix: /api/hello
          queryParams:
            version:
              exact: v1
      route:
        - destination:
            host: hello-service
            subset: v1
            port:
              number: 8080
    - match:
        - uri:
            prefix: /api/hello
          queryParams:
            version:
              exact: v2
      route:
        - destination:
            host: hello-service
            subset: v2
            port:
              number: 8080
    - match:
        - uri:
            prefix: /api/hello
          queryParams:
            version:
              exact: v3
      route:
        - destination:
            host: hello-service
            subset: v3
            port:
              number: 8080
    - retries:
        attempts: 3
        perTryTimeout: 1s
        retryOn: 5xx
      route:
        - destination:
            host: hello-service
            subset: default
            port:
              number: 8080
```

