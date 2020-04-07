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

