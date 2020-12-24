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



## HTTP 重定向

HTTP重定向（HTTP Redirect）能够让单个页面、表单或者整个Web应用都跳转到新的 URL 下，该 操作可以应用于多种场景：网站维护期间的临时跳转，网站架构改变后为了保持外部链接继续可用的永 久重定向，上传文件时的进度页面等。

修改 hello 的 VirtualService ：

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
      name: http-haha
    - match:
        - uri:
            prefix: /api/hello
          queryParams:
            version:
              exact: v2
      redirect:
        authority: fueltank-1
        uri: /api/hello?version=v3
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

查看其中的 `redirect` 配置。

访问：

```bash
$ curl http://fueltank-1/api/hello?version=v2 -v
```

也可以浏览器测试结果



## HTTP 重写

继续修改 ：

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
      name: http-haha
    - match:
        - uri:
            prefix: /api/he
      rewrite:
        uri: /api/hello
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

注意看其中的 `rewrite` 配置。

访问：

```bash
$ curl http://fueltank-1/api/he
```



## 熔断

服务端的Proxy会记录调用发生错误的次数，然后根据配置决定是否继续提供服务或者立刻返回错 误。使用熔断机制可以保护服务后端不会过载。

## 限流

限流是一种预防措施，在发生灾难发生前就对并发访问进行限制。Istio的速率限制特性可以实现常见 的限流功能，即防止来自外部服务的过度调用。衡量指标主要是QPS（每秒请求量），实现方式是计数器 方式。Istio 支持 Memquota 适配器和 Redisquota 适配器。

原理：https://istio.io/latest/zh/docs/tasks/policy-enforcement/rate-limiting/#understanding-rate-limits

先前的例子中你已经看到了 Mixer 是怎么通过匹配特定的条件对请求应用速率限制的。

每个具名的 quota 实例比如 `requestcount` 会提供一组计数器。 这组计数器用所有 quota 维度的笛卡尔积来定义。如果最后一个`有效期`内的请求数超出 `maxAmount`, Mixer 返回一个 `RESOURCE_EXHAUSTED` 消息给 Envoy 代理，然后 Envoy 返回状态码 `HTTP 429` 给调用者。

`memquota` 适配器使用一个亚秒级的滑动窗口来执行速率限制。

`redisquota` 适配器可以配置使用 [`ROLLING_WINDOW` 或 `FIXED_WINDOW`](https://istio.io/latest/zh/docs/reference/config/policy-and-telemetry/adapters/redisquota/#Params-QuotaAlgorithm) 算法之一来执行速率限制。

适配器配置内的 `maxAmount` 为所有关联到 quota 实例的计数器设置了默认限制。这个默认限制应用在其他优先规则没有被匹配到的时候。 `memquota/redisquota` 适配器会选择第一个与请求相匹配的优先规则。一个优先规则不能指明所有的 quota 维度。 在这个例子里，0.2 qps 的规则被选择到通过只匹配了四分之三的 quota 维度。

如果您想要策略在给定的命名空间上执行而非整个 Istio 网格，可以把前面所有出现的 `istio-system` 替换为您想要的命名空间。

限流算法： [三种限流算法.md](../../../其他/算法/三种限流算法.md) 

## 服务隔离

即 Sidecar 资源的使用方式。

## 影子测试

查找新代码错误的最佳方法是在实际环境中进行测试。影子测试可以将生产流量复制到目标服务中 进行测试，在处理中产生的任何错误都不会对整个系统的性能和可靠性造成影响。 

配置：

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
      mirror:
        host: hello-service
        subset: v2
      route:
        - destination:
            host: hello-service
            subset: v1
            port:
              number: 8080
```

上面配置的策略将全部流量都发送到 hello-service 服务的 v1 版本，其中的 mirror 字段指定将流量复制到  forecast 服务的v2版本。当流量被复制时，会在请求的HOST或Authority头中添加-shadow后缀（例如 hello-service-shadow）并将请求发送到 hello-service 服务的v2版本以示它是影子流量。这些被复制的请求引发的响应 会被丢弃，不会影响终端客户。 

