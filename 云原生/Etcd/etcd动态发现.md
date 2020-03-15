# etcd 动态发现

我在  [Kubernetes二进制安装.md](../Kubernetes/Kubernetes二进制安装.md) 中使用了静态发现来创建了 etcd 集群。下面来学习一下如何通过动态发现来创建 etcd 集群。

官网教程：https://etcd.io/docs/v3.4.0/dev-internal/discovery_protocol/

废话不多说，来动手：

```bash
$ curl https://discovery.etcd.io/new?size=3	#使用公共etcd发现服务
https://discovery.etcd.io/c41a8c03922c32eaa4be1e7bcf633123
```

然后在 etcd 配置中，干掉 `ETCD_INITIAL_CLUSTER` 这个环境变量来干掉静态发现服务，然后配置 `ETCD_DISCOVERY=` 为 `https://discovery.etcd.io/c41a8c03922c32eaa4be1e7bcf63123` 就搞定了。

在启动 etcd 时，最好多台机器一同启动。

查看集群成员：

```bash
$ etcdctl member list
```



如果想自己搭建私有的动态发现服务，可以使用：https://github.com/coreos/discovery.etcd.io