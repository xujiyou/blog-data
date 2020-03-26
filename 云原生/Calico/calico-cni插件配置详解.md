# calico cni 插件配置详解

官方文档地址：https://docs.projectcalico.org/reference/cni-plugin/configuration

calico cni 插件的配置文件是 `cat /etc/cni/net.d/10-calico.conflist`

我的配置如下：

```json
{
  "name": "k8s-pod-network",
  "cniVersion": "0.3.1",
  "plugins": [
    {
      "type": "calico",
      "log_level": "info",
      "etcd_endpoints": "https://fueltank-1:2379,https://fueltank-2:2379,https://fueltank-3:2379",
      "etcd_key_file": "/etc/cni/net.d/calico-tls/etcd-key",
      "etcd_cert_file": "/etc/cni/net.d/calico-tls/etcd-cert",
      "etcd_ca_cert_file": "/etc/cni/net.d/calico-tls/etcd-ca",
      "mtu": 1440,
      "ipam": {
          "type": "calico-ipam"
      },
      "policy": {
          "type": "k8s"
      },
      "kubernetes": {
          "kubeconfig": "/etc/cni/net.d/calico-kubeconfig"
      }
    },
    {
      "type": "portmap",
      "snat": true,
      "capabilities": {"portMappings": true}
    },
    {
      "type": "bandwidth",
      "capabilities": {"bandwidth": true}
    }
  ]
}
```



## 数据保存类型

字段 `datastore_type` 表示储存类型，默认为 `etcdv3` ，calico 支持两种类型：etcdv3 和 kubernetes



## etcd 配置

`etcd_endpoints` 用来配置 etcd 的静态发现的地址列表

`etcd_discovery_srv` 用来配置 etcd 动态服务发现的地址列表

`etcd_key_file` 用来配置 etcd 的私钥

`etcd_cert_file` 用来配置 etcd 的证书

`etcd_ca_cert_file` 用来配置 etcd 的 CA 文件



## 日志

- 记录总是 `stderr`
- 可以通过 `log_level` 在netconf中进行设置来控制日志记录级别。允许的级别是
  - `ERROR` -仅发出错误日志。
  - `WARNING` -默认值。
  - `INFO` -从CNI插件启用一些其他日志记录。
  - `DEBUG` -从CNI插件和基础libcalico库启用大量调试日志记录。



## IPAM

`assign_ipv4` 默认为 `true`，`assign_ipv6` 默认为 `false`

可以使用[`CNI_ARGS`](https://github.com/appc/cni/blob/master/SPEC.md#parameters)并将其设置`IP`为所需的值来选择特定的IP地址。

默认情况下，Calico IPAM将分配所有可用IP池中的IP地址。

（可选）还可以通过以下属性指定可能的IPv4和IPv6池的列表：

- `ipv4_pools`：CIDR字符串或池名称的数组。（例如，`"ipv4_pools": ["10.0.0.0/24", "20.0.0.0/16", "default-ipv4-ippool"]`）
- `ipv6_pools`：CIDR字符串或池名称的数组。（例如，`"ipv6_pools": ["2001:db8::1/120", "namedpool"]`）

在我的配置中，ipam 中只有一个字段 "type": "calico-ipam" ，查看 `/opt/cni/bin/` 中会发现一个 `calico-ipam` 的二进制文件。



## 容器设置

以下选项允许在容器名称空间中配置设置。

- allow_ip_forwarding（默认为`false`）

```json
{
    "name": "any_name",
    "cniVersion": "0.1.0",
    "type": "calico",
    "ipam": {
        "type": "calico-ipam"
    },
    "container_settings": {
        "allow_ip_forwarding": true
    }
}
```



## Kubernetes

将Calico CNI插件与Kubernetes一起使用时，该插件必须能够访问Kubernetes API服务器，以查找分配给 Pod 的标签。推荐的配置访问方式是通过网络配置部分中`kubeconfig`指定的文件`kubernetes`。例如

```json
{
    "name": "any_name",
    "cniVersion": "0.1.0",
    "type": "calico",
    "kubernetes": {
        "kubeconfig": "/path/to/kubeconfig"
    },
    "ipam": {
        "type": "calico-ipam"
    }
}
```

为了方便起见，API位置也可以直接配置，例如

```json
{
    "name": "any_name",
    "cniVersion": "0.1.0",
    "type": "calico",
    "kubernetes": {
        "k8s_api_root": "http://127.0.0.1:8080"
    },
    "ipam": {
        "type": "calico-ipam"
    }
}
```



## 启用 Kubernetes 策略

如果希望使用Kubernetes `NetworkPolicy`资源，则必须在网络配置中设置策略类型。有一个受支持的策略类型，`k8s`。设置后，还必须在启用了策略，配置文件和工作负载端点控制器的情况下运行calico / kube-controllers。

```json
{
    "name": "any_name",
    "cniVersion": "0.1.0",
    "type": "calico",
    "policy": {
      "type": "k8s"
    },
    "kubernetes": {
        "kubeconfig": "/path/to/kubeconfig"
    },
    "ipam": {
        "type": "calico-ipam"
    }
}
```

使用时`type: k8s`，Calico CNI插件要求`Pods`对所有名称空间中的资源具有只读的Kubernetes API访问权限。



### CNI网络配置列表

CNI 0.3.0 [规范](https://github.com/containernetworking/cni/blob/spec-v0.3.0/SPEC.md#network-configuration-lists)支持将多个cni插件“链接”在一起，而Calico也支持此功能。Calico默认情况下启用portmap插件，这是实现Kubernetes主机端口功能所必需的。可以通过从Calico清单的CNI网络配置中删除portmap部分来禁用此功能。

```
        {
          "type": "portmap",
          "snat": true,
          "capabilities": {"portMappings": true}
        }
```

> **注意**：portmap插件存在CNI问题，在此情况下，包含100多个节点和4000多个服务的群集的排空节点可能需要很长时间。参见https://github.com/containernetworking/cni/issues/605