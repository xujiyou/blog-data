# Node

官方文档：https://docs.projectcalico.org/reference/resources/node

节点资源（`Node`）代表运行Calico的节点。将主机添加到Calico群集时，需要创建一个节点资源，其中包含`calico/node`在主机上运行的实例的配置。

启动`calico/node`实例时，提供给该实例的名称应与Node资源中配置的名称匹配。

默认情况下，启动`calico/node`实例将使用`hostname`计算主机的来自动创建节点资源。

示例：

```yaml
apiVersion: projectcalico.org/v3
kind: Node
metadata:
  name: node-hostname
spec:
  bgp:
    asNumber: 64512
    ipv4Address: 10.244.0.1/24
    ipv6Address: 2001:db8:85a3::8a2e:370:7334/120
    ipv4IPIPTunnelAddr: 192.168.0.1
```



#### Spec

| Field               | Description                                         | Accepted Values | Schema                                                       | Default |
| :------------------ | :-------------------------------------------------- | :-------------- | :----------------------------------------------------------- | :------ |
| bgp                 | 该节点的BGP配置。如果仅将Calico用于保单，则省略。   |                 | [BGP](https://docs.projectcalico.org/reference/resources/node#bgp) |         |
| ipv4VXLANTunnelAddr | VXLAN隧道的IPv4地址。这是系统配置的，不应手动更新。 |                 | string                                                       |         |
| vxlanTunnelMACAddr  | VXLAN隧道的MAC地址。这是系统配置的，不应手动更新。  |                 | string                                                       |         |
| orchRefs            | 将此节点与另一个协调器中的节点关联。                |                 | list of [OrchRefs](https://docs.projectcalico.org/reference/resources/node#OrchRef) |         |



#### BGP

| Field                   | Description                                            | Accepted Values                                              | Schema  | Default |
| :---------------------- | :----------------------------------------------------- | :----------------------------------------------------------- | :------ | :------ |
| asNumber                | 您的的AS号`calico/node`。                              | Optional. If omitted the global value is used (see [example modifying Global BGP settings](https://docs.projectcalico.org/networking/bgp) for details about modifying the `asNumber` setting). | integer |         |
| ipv4Address             | 导出为主机上Calico端点的下一跳的IPv4地址和子网         | The IPv4 address must be specified if BGP is enabled.        | string  |         |
| ipv6Address             | 导出为主机上Calico端点的下一跳的IPv6地址和子网         | Optional                                                     | string  |         |
| ipv4IPIPTunnelAddr      | IP-in-IP隧道的IPv4地址。这是系统配置的，不应手动更新。 | Optional IPv4 address                                        | string  |         |
| routeReflectorClusterID | 在给定群集中将此节点启用为路由反射器                   | Optional IPv4 address                                        | string  |         |