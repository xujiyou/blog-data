# FelixConfiguration

官方文档：https://docs.projectcalico.org/reference/resources/felixconfig

集群获取结果：

```bash
$ calicoctl get FelixConfiguration
NAME              
default           
node.fueltank-1   
node.fueltank-2   
node.fueltank-3
```

详细信息：

```yaml
$ calicoctl get FelixConfiguration default -o yaml
apiVersion: projectcalico.org/v3
kind: FelixConfiguration
metadata:
  creationTimestamp: "2020-03-23T02:46:12Z"
  name: default
  resourceVersion: "573849"
  uid: 1b951893-6553-4bd5-a516-1f85eb3af171
spec:
  bpfLogLevel: ""
  ipipEnabled: true
  logSeverityScreen: Info
  policySyncPathPrefix: /var/run/nodeagent
  reportingInterval: 0s
  
$ calicoctl get FelixConfiguration node.fueltank-1  -o yaml
apiVersion: projectcalico.org/v3
kind: FelixConfiguration
metadata:
  creationTimestamp: "2020-03-23T02:46:33Z"
  name: node.fueltank-1
  resourceVersion: "475"
  uid: 06181ca7-2775-44fd-82e2-88730058f6b0
spec:
  bpfLogLevel: ""
  defaultEndpointToHostAction: Return
```



简单示例：

```yaml
apiVersion: projectcalico.org/v3
kind: FelixConfiguration
metadata:
  name: default
spec:
  ipv6Support: false
  ipipMTU: 1400
  chainInsertMode: Append
```



Spec 中可配置的字段还挺多的。

| Field                       | Description                                                  | Accepted Values      | Schema                                                       | Default                 |
| :-------------------------- | :----------------------------------------------------------- | :------------------- | :----------------------------------------------------------- | :---------------------- |
| chainInsertMode             | 表示calico规则插入 iptables 的方式，Insert是插入，可以保证 calico规则不被忽略，Append 方式是在最后追加，规则有可能被忽略 | Insert, Append       | string                                                       | `Insert`                |
| defaultEndpointToHostAction | 这个参数会控制从 endpoint 到主机的流量，Drop是阻止所有流量，如果要允许，请使用 Retuen 或 Accept，Retuen 会在处理完成后处理自定义规则，Accept 则表示全部接受 | Drop, Return, Accept | string                                                       | `Drop`                  |
| deviceRouteSourceAddress    | 设置为Felix编程的路由的源提示的IPv4地址。如果未设置，则从主机到工作负载的本地流量的源地址将由内核确定 | IPv4                 | string                                                       | ""                      |
| deviceRouteProtocol         | 这定义了添加到已编程设备路由的路由协议。                     | Protocol             | int                                                          | RTPROT_BOOT             |
| failsafeInboundHostPorts    | Felix将允许UDP / TCP / SCTP协议/端口对，无论安全策略如何，传入流量都可以将其承载到主机端点。这有助于避免意外切断配置错误的主机。默认值允许SSH访问，etcd，BGP和DHCP。 |                      | List of [ProtoPort](https://docs.projectcalico.org/reference/resources/felixconfig#protoport) | Tcp:22,udp:68,tcp:179等 |
| failsafeOutboundHostPorts   | 允许的传出流量的配置                                         |                      | List of [ProtoPor](https://docs.projectcalico.org/reference/resources/felixconfig#protoport) | Udp:53 等               |
| genericXDPEnabled           | 启用后，Felix可以回`generic`退到非优化的XDP模式。这只能用于测试，因为它不会比非XDP模式提高性能。 | true,false           | boolean                                                      | false                   |
| ignoreLooseRPF              |                                                              |                      |                                                              |                         |
|                             |                                                              |                      |                                                              |                         |