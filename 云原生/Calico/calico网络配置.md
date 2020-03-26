# calico 配置详解

官方关于 calico 的配置文档：https://docs.projectcalico.org/networking/configuring

这里先学习下 overlay 的配置，我的集群不支持 BGP，无法实验，所以先不管 BGP 模式了。

calico 的配置主要是使用 FelixConfiguration 这个资源对象来实现的。具体查看： [FelixConfiguration.md](资源对象/FelixConfiguration.md) 

查看集群中使用的模式及IP池：

```yaml
$ calicoctl get ipPool -o yaml
apiVersion: projectcalico.org/v3
items:
- apiVersion: projectcalico.org/v3
  kind: IPPool
  metadata:
    creationTimestamp: "2020-03-23T02:46:13Z"
    name: default-ipv4-ippool
    resourceVersion: "457"
    uid: 1a09ec6f-cc01-4b57-9d1f-d43cf3ef0a97
  spec:
    blockSize: 26
    cidr: 10.42.0.0/16
    ipipMode: Always
    natOutgoing: true
    nodeSelector: all()
    vxlanMode: Never
kind: IPPoolList
metadata:
  resourceVersion: "541988"
```

可以看出，我的集群使用的是 IPIP 模式的。

