# GlobalNetworkSet

官方文档：https://docs.projectcalico.org/reference/resources/globalnetworkset

GlobalNetworkSet 就是一个 IP子网/ CIDR的任意集合，从而可以让 `GlobalNetworkPolicy` 或 `NetworkPolicy` 通过标签来选择地址集。

`NetworkSet` 是同样的套路，只不过加了一个 namespace 字段而已。

例子：

```yaml
apiVersion: projectcalico.org/v3
kind: GlobalNetworkSet
metadata:
  name: a-name-for-the-set
  labels:
    role: external-database
spec:
  nets:
  - 198.51.100.0/28
  - 203.0.113.0/24
```

很简单，只有一个 nets 字段。