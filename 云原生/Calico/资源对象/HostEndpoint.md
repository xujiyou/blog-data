# HostEndpoint

官方文档：https://docs.projectcalico.org/reference/resources/hostendpoint

物理主机的地址

示例：

```yaml
apiVersion: projectcalico.org/v3
kind: HostEndpoint
metadata:
  name: some.name
  labels:
    type: production
spec:
  interfaceName: eth0
  node: myhost
  expectedIPs:
  - 192.168.0.1
  - 192.168.0.2
  profiles:
  - profile1
  - profile2
  ports:
  - name: some-port
    port: 1234
    protocol: TCP
  - name: another-port
    port: 5432
    protocol: UDP
```



## spec

#### Spec

| Field         | Description                    | Accepted Values            | Schema                                                       | Default |
| :------------ | :----------------------------- | :------------------------- | :----------------------------------------------------------- | :------ |
| node          | 主机名                         |                            | string                                                       |         |
| interfaceName | 物理网口                       |                            | string                                                       |         |
| expectedIPs   | 与接口关联的IP地址。           | Valid IPv4 or IPv6 address | list                                                         |         |
| profiles      | 要应用于端点的配置文件列表。   |                            | list                                                         |         |
| ports         | 此工作负荷公开的命名端口列表。 |                            | List of [EndpointPorts](https://docs.projectcalico.org/reference/resources/hostendpoint#endpointport) |         |