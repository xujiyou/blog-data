# GlobalNetworkPolicy

`GlobalNetworkPolicy` 是没有命名空间的限制的，而 `NetworkPolicy` 是有命名空间的。

`NetworkPolicy` 只比 `GlobalNetworkPolicy` 在 Metadata 那里多了一个命名空间。

`Profile` 也是一样的套路，只不过是用于主机的规则，而 `NetworkPolicy` 和 `GlobalNetworkPolicy` 是用于 Pod 的规则。

示例：

```yaml
apiVersion: projectcalico.org/v3
kind: GlobalNetworkPolicy
metadata:
  name: allow-tcp-6379
spec:
  selector: role == 'database'
  types:
  - Ingress
  - Egress
  ingress:
  - action: Allow
    metadata:
      annotations:
        from: frontend
        to: database
    protocol: TCP
    source:
      selector: role == 'frontend'
    destination:
      ports:
      - 6379
  egress:
  - action: Allow
```



## spec

| Field                  | Description                                                  | Accepted Values   | Schema                                                       | Default                    |
| :--------------------- | :----------------------------------------------------------- | :---------------- | :----------------------------------------------------------- | :------------------------- |
| order                  | 控制优先顺序。 Calico首先应用最低值的策略。                  |                   | float                                                        |                            |
| selector               | 选择此策略适用的 endpoints                                   |                   | [selector](https://docs.projectcalico.org/reference/resources/globalnetworkpolicy#selector) | all()                      |
| serviceAccountSelector | 选择此策略适用的服务帐户。使用`projectcalico.org/name`标签选择具有特定名称的群集中的所有服务帐户。 |                   | [selector](https://docs.projectcalico.org/reference/resources/globalnetworkpolicy#selector) | all()                      |
| namespaceSelector      | 选择此策略适用的名称空间。使用`projectcalico.org/name`标签通过名称选择特定的名称空间。 |                   | [selector](https://docs.projectcalico.org/reference/resources/globalnetworkpolicy#selector) | all()                      |
| types                  | 根据流量方向应用策略。要将策略应用于入站流量，请设置为`Ingress`。要将策略应用于出站流量，请设置为`Egress`。要将策略应用于两者，请设置为`Ingress, Egress`。 | Ingress`, `Egress | List of strings                                              | 取决于入口/出口规则的存在* |
| ingress                | 策略应用的入口规则的有序列表。                               |                   | List of [Rule](https://docs.projectcalico.org/reference/resources/globalnetworkpolicy#rule) |                            |
| egress                 | 策略应用的出口规则的有序列表。                               |                   | List of [Rule](https://docs.projectcalico.org/reference/resources/globalnetworkpolicy#rule) |                            |
| doNotTrack**           | 指示在任何数据平面连接跟踪之前应用此策略中的规则，并且不应跟踪这些规则允许的数据包。 | true, false       | boolean                                                      | false                      |
| preDNAT**              | 表示在任何DNAT之前应用此策略中的规则。                       | true, false       | boolean                                                      | false                      |
| applyOnForward**       | 表示将此策略中的规则应用于转发的流量以及本地终止的流量。     | true, false       | boolean                                                      | false                      |

*如果`types`没有值，则Calico的默认设置如下。

> | 存在入口规则 | 存在出口规则 | `Types` 值        |
> | :----------- | :----------- | :---------------- |
> | 没有         | 没有         | `Ingress`         |
> | 是           | 没有         | `Ingress`         |
> | 没有         | 是           | `Egress`          |
> | 是           | 是           | `Ingress, Egress` |

`doNotTrack`和`preDNAT`和`applyOnForward`字段仅在将策略应用于[主机端点](https://docs.projectcalico.org/reference/resources/hostendpoint)时才有意义。



## Rule

| Field       | Description                                              | Accepted Values                                            | Schema                                                       | Default |
| :---------- | :------------------------------------------------------- | :--------------------------------------------------------- | :----------------------------------------------------------- | :------ |
| metadata    | 每条规则的元数据                                         |                                                            | [RuleMetadata](https://docs.projectcalico.org/reference/resources/globalnetworkpolicy#rulemetadata) |         |
| action      | 匹配此规则时要执行的操作。                               | Allow`, `Deny`, `Log`, `Pass                               | string                                                       |         |
| protocol    | 要匹配的协议                                             | TCP`, `UDP`, `ICMP`, `ICMPv6`, `SCTP`, `UDPLite`, `1`-`255 | string 或 int                                                |         |
| notProtocol | 不匹配的协议                                             | TCP`, `UDP`, `ICMP`, `ICMPv6`, `SCTP`, `UDPLite`, `1`-`255 | string 或 int                                                |         |
| icmp        | ICMP匹配条件。                                           |                                                            | [ ICMP](https://docs.projectcalico.org/reference/resources/globalnetworkpolicy#icmp) |         |
| notICMP     | ICMP上的否定匹配。                                       |                                                            | [ICMP](https://docs.projectcalico.org/reference/resources/globalnetworkpolicy#icmp) |         |
| ipVersion   | ip 版本                                                  | 4`, `6                                                     | int                                                          |         |
| source      | 源                                                       |                                                            | [EntityRule](https://docs.projectcalico.org/reference/resources/globalnetworkpolicy#entityrule) |         |
| destination | 目的地                                                   |                                                            | [EntityRule](https://docs.projectcalico.org/reference/resources/globalnetworkpolicy#entityrule) |         |
| http        | 匹配HTTP请求参数。必须启用应用程序层策略才能使用此字段。 |                                                            | [HTTP匹配](https://docs.projectcalico.org/reference/resources/globalnetworkpolicy#httpmatch) |         |



#### RuleMetadata

与特定规则（而不是整个策略）关联的元数据。元数据的内容不影响规则的解释或执行方式。它只是一种存储其他信息以供与Calico进行交互的操作员或应用程序使用的方法。



#### ICMP

| Field | Description         | Accepted Values      | Schema  | Default |
| :---- | :------------------ | :------------------- | :------ | :------ |
| type  | Match on ICMP type. | Can be integer 0-254 | integer |         |
| code  | Match on ICMP code. | Can be integer 0-255 | integer |         |



#### EntityRule

| Field             | Description                                                  | Accepted Values                          | Schema                                                       | Default |
| :---------------- | :----------------------------------------------------------- | :--------------------------------------- | :----------------------------------------------------------- | :------ |
| nets              | 匹配任何列出的CIDR中具有IP的数据包。                         | 有效的IPv4 CIDR列表或有效的IPv6 CIDR列表 | list of cidrs                                                |         |
| notNets           | CIDR上的否定匹配                                             | 有效的IPv4 CIDR列表或有效的IPv6 CIDR列表 | list of cidrs                                                |         |
| selector          | 选定端点上的正匹配。如果还定义了`namespaceSelector` ，则该端点适用于所选命名空间中的端点。 | Valid selector                           | [selector](https://docs.projectcalico.org/reference/resources/globalnetworkpolicy#selector) |         |
| notSelector       | 选定端点上的反匹配                                           | Valid selector                           | [selector](https://docs.projectcalico.org/reference/resources/globalnetworkpolicy#selector) |         |
| namespaceSelector | 所选名称空间上的正匹配。如果指定，则仅匹配所选Kubernetes名称空间中的工作负载端点。根据已应用于名称空间的标签来匹配名称空间。定义选择器将应用于的上下文，如果未定义，则选择器将应用于NetworkPolicy的名称空间。使用`projectcalico.org/name`标签通过名称匹配特定的名称空间。 | Valid selector                           | [selector](https://docs.projectcalico.org/reference/resources/globalnetworkpolicy#selector) |         |
| ports             | 指定端口上的正匹配                                           |                                          | list of [ports](https://docs.projectcalico.org/reference/resources/globalnetworkpolicy#ports) |         |
| notPorts          | 指定端口上的反匹配                                           |                                          | list of [ports](https://docs.projectcalico.org/reference/resources/globalnetworkpolicy#ports) |         |
| serviceAccounts   | 匹配在服务帐户下运行的端点。如果还定义了`namespaceSelector` ，则适用于该服务帐户的集合仅限于所选名称空间中的服务帐户。 |                                          | [ServiceAccountMatch](https://docs.projectcalico.org/reference/resources/globalnetworkpolicy#serviceaccountmatch) |         |



#### Selector

标签选择器是根据其标签匹配或不匹配资源的表达式。

印花布标签选择器支持多种语法原语。以下每个基本表达式都可以使用逻辑运算符`&&`和进行组合`||`。

| Syntax                  | Meaning                                                      |
| :---------------------- | :----------------------------------------------------------- |
| all()                   | Match all resources.                                         |
| k == ‘v’                | Matches any resource with the label ‘k’ and value ‘v’.       |
| k != ‘v’                | Matches any resource with the label ‘k’ and value that is *not* ‘v’. |
| has(k)                  | Matches any resource with label ‘k’, independent of value.   |
| !has(k)                 | Matches any resource that does not have label ‘k’            |
| k in { ‘v1’, ‘v2’ }     | Matches any resource with label ‘k’ and value in the given set |
| k not in { ‘v1’, ‘v2’ } | Matches any resource without label ‘k’ or any with label ‘k’ and value *not* in the given set |
| k contains ‘s’          | Matches any resource with label ‘k’ and value containing the substring ‘s’ |
| k starts with ‘s’       | Matches any resource with label ‘k’ and value starting with the substring ‘s’ |
| k ends with ‘s’         | Matches any resource with label ‘k’ and value ending with the substring ‘s’ |



#### Ports

Calico supports the following syntaxes for expressing ports.

| Syntax    | Example    | Description                                                  |
| :-------- | :--------- | :----------------------------------------------------------- |
| int       | 80         | The exact (numeric) port specified                           |
| start:end | 6040:6050  | All (numeric) ports within the range start <= x <= end       |
| string    | named-port | A named port, as defined in the ports list of one or more endpoints |

例子：

```
ports: [8080, "1234:5678", "named-port"]
```



#### ServiceAccountMatch

A ServiceAccountMatch matches service accounts in an EntityRule.

| Field    | Description                     | Schema                                                       |
| :------- | :------------------------------ | :----------------------------------------------------------- |
| names    | Match service accounts by name  | list of strings                                              |
| selector | Match service accounts by label | [selector](https://docs.projectcalico.org/reference/resources/globalnetworkpolicy#selector) |



#### HTTPMatch

HTTPMatch与HTTP请求的属性匹配。规则上存在HTTPMatch子句将导致该规则仅匹配HTTP流量。其他应用程序层协议将不符合该规则。

Example:

```
http:
  methods: ["GET", "PUT"]
  paths:
    - exact: "/projects/calico"
    - prefix: "/users"
```