# WorkloadEndpoint

官方文档：https://docs.projectcalico.org/reference/resources/workloadendpoint

工作负载端点资源（`WorkloadEndpoint`）表示将Calico网络容器或VM连接到其主机的接口。

每个端点可以指定一组标签和配置文件列表，Calico将使用这些标签和配置文件列表将策略应用于接口。

工作负载端点是命名空间资源，这意味着 特定命名空间中的 [NetworkPolicy](https://docs.projectcalico.org/reference/resources/networkpolicy)仅适用于该命名空间中的WorkloadEndpoint。如果两个资源的名称空间值设置相同，则两个资源位于同一名称空间中。

这个资源通常是 calico 给生成的。

示例：

```yaml
apiVersion: projectcalico.org/v3
kind: WorkloadEndpoint
metadata:
  name: node1-k8s-my--nginx--b1337a-eth0
  namespace: default
  labels:
    app: frontend
    projectcalico.org/namespace: default
    projectcalico.org/orchestrator: k8s
spec:
  node: node1
  orchestrator: k8s
  endpoint: eth0
  containerID: 1337495556942031415926535
  pod: my-nginx-b1337a
  endpoint: eth0
  interfaceName: cali0ef24ba
  mac: ca:fe:1d:52:bb:e9
  ipNetworks:
  - 192.168.0.0/32
  profiles:
  - profile1
  ports:
  - name: some-port
    port: 1234
    protocol: TCP
  - name: another-port
    port: 5432
    protocol: UDP
```



#### Spec

| Field         | Description                                     | Accepted Values | Schema                                                       | Default |
| :------------ | :---------------------------------------------- | :-------------- | :----------------------------------------------------------- | :------ |
| workload      | 该端点所属的工作负载的名称。                    |                 | string                                                       |         |
| orchestrator  | 创建此端点的协调器。                            |                 | string                                                       |         |
| node          | 该端点所在的节点。                              |                 | string                                                       |         |
| containerID   | CNI 容器 ID                                     |                 | string                                                       |         |
| pod           | Kubernetes pod name for this woekload endpoint. |                 | string                                                       |         |
| endpoint      | 容器网络接口名称。                              |                 | string                                                       |         |
| ipNetworks    | 分配给接口的CIDR。                              |                 | List of strings                                              |         |
| ipNATs        | 适用于端点的1：1 NAT映射列表                    |                 | List of [IPNATs](https://docs.projectcalico.org/reference/resources/workloadendpoint#ipnat) |         |
| ipv4Gateway   | 来自工作负载的流量的网关IPv4地址。              |                 | string                                                       |         |
| ipv6Gateway   | 来自工作负载的流量的网关IPv6地址。              |                 | string                                                       |         |
| profiles      | 分配给该端点的配置文件列表。                    |                 | List of strings                                              |         |
| interfaceName | 附加到工作负载的主机侧接口的名称。              |                 | string                                                       |         |
| mac           | 工作负载生成的流量的源MAC地址。                 |                 | IEEE 802 MAC-48, EUI-48, or EUI-64                           |         |
| ports         | 在此工作负载公开的命名端口上列出。              |                 | List of [EndpointPorts](https://docs.projectcalico.org/reference/resources/workloadendpoint#endpointport) |         |