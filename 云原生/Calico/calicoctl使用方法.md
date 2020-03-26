# calicoctl 使用方法

安装方法：https://docs.projectcalico.org/getting-started/calicoctl/install

我是在本地主机上安装的，下面是我的配置(自己创建的文件)：

```yaml
$ cat /etc/calico/calicoctl.cfg
apiVersion: projectcalico.org/v3
kind: CalicoAPIConfig
metadata:
spec:
  etcdEndpoints: https://fueltank-1:2379,https://fueltank-2:2379,https://fueltank-3:2379
  etcdKeyFile: /etc/etcd/cert/etcd/etcd-key.pem
  etcdCertFile: /etc/etcd/cert/etcd/etcd.pem
  etcdCACertFile: /etc/etcd/cert/etcd/ca.pem
```

calico cni 的配置文件(calico Pod 自己创建的)：

```json
$ cat /etc/cni/net.d/10-calico.conflist 
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



我当前使用的版本：

```bash
$ calicoctl version
Client Version:    v3.13.0
Git commit:        eb796e31
Cluster Version:   v3.13.0
Cluster Type:      k8s,bgp,kubeadm
```



calicoctl 一共有 11 个子命令，下面分别学习。



## create

与 `kubectl create` 类似， calicoctl 也能创建资源。

与之类似的子命令还有 `replace`、`apply` 、`patch` 、`delete` 等



## get

帮助信息通过 `calicoctl get -h` 查看。

可以获取的资源对象包括：

* bgpConfiguration
* bgpPeer
* felixConfiguration
* globalNetworkPolicy
* globalNetworkSet
* hostEndpoint
* ipPool
* networkPolicy
* networkSet
* node
* profile
* workloadEndpoint

其中，NetworkPolicy, NetworkSet, and WorkloadEndpoint 是有命名空间的。

输出格式包括：

yaml, json, ps, wide, custom-columns=..., go-template=...,go-template-file=...



## label

可以更新 get 中提到的资源列表的 label



## convert

在不同的 API 之间转换



## ipam

查看 calico 的 IP地址分配情况：

```
$ calicoctl ipam show
+----------+--------------+-----------+------------+--------------+
| GROUPING |     CIDR     | IPS TOTAL | IPS IN USE |   IPS FREE   |
+----------+--------------+-----------+------------+--------------+
| IP Pool  | 10.42.0.0/16 |     65536 | 30 (0%)    | 65506 (100%) |
+----------+--------------+-----------+------------+--------------+
```



## node

查看主机信息

诊断：

```bash
$ sudo calicoctl node diags
```

查看主机状态：

```bash
$ sudo calicoctl node status
```

系统特性检查：

```bash
$ sudo calicoctl node checksystem
```

