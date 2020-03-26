# IPPool

官方文档：https://docs.projectcalico.org/reference/resources/ippool

IP池资源（`IPPool`）代表IP地址的集合，Calico希望从这些IP地址中为 Pod 分配端点IP。

示例：

```yaml
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
  name: my.ippool-1
spec:
  cidr: 10.1.0.0/16
  ipipMode: CrossSubnet
  natOutgoing: true
  disabled: false
  nodeSelector: all()
```



## spec

#### Spec

| Field        | Description                                                  | Accepted Values                                              | Schema                                                       | Default                                       |
| :----------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :-------------------------------------------- |
| cidr         | 地址池                                                       | 有效的IPv4或IPv6 CIDR。子网长度必须至少足够大以适合单个块（`/26`对于IPv4或`/122`IPv6 ，默认情况下）。不得与“链接本地”范围`169.254.0.0/16`或重叠`fe80::/10`。 | string                                                       |                                               |
| blockSize    | 此池使用的分配块的CIDR大小。块按需分配给主机，并用于汇总路由。该值只能在创建池时设置。 | IPv4为20至32（含），IPv6为116至128（含）                     | int                                                          | `26` for IPv4 pools and `122` for IPv6 pools. |
| ipipMode     | 定义使用IPIP的模式。不能与同时设置`vxlanMode`。              | Always, CrossSubnet, Never                                   | string                                                       | `Never`                                       |
| vxlanMode    | 定义使用VXLAN的模式。不能与同时设置`ipipMode`。              | Always, CrossSubnet, Never                                   | string                                                       | `Never`                                       |
| natOutgoing  | 启用后，将从该池中的Calico网络容器发送到该池之外的目的地的数据包伪装。 | true, false                                                  | boolean                                                      | `false`                                       |
| disabled     | 设置为true时，Calico IPAM将不会分配该池中的地址。            | true, false                                                  | boolean                                                      | `false`                                       |
| nodeSelector | 选择Calico IPAM应该从该池中分配地址的节点。                  |                                                              | [selector](https://docs.projectcalico.org/reference/resources/ippool#node-selector) | all()                                         |