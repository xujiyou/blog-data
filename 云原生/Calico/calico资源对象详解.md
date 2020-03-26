# Calico 资源对象详解



## FelixConfiguration

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

| Field           | Description                                                  | Accepted Values | Schema | Default  |
| :-------------- | :----------------------------------------------------------- | :-------------- | :----- | :------- |
| chainInsertMode | Controls whether Felix hooks the kernel’s top-level iptables chains by inserting a rule at the top of the chain or by appending a rule at the bottom. `Insert` is the safe default since it prevents Calico’s rules from being bypassed. If you switch to `Append` mode, be sure that the other rules in the chains signal acceptance by falling through to the Calico rules, otherwise the Calico policy will be bypassed. | Insert, Append  | string | `Insert` |