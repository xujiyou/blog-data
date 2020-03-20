# calicoctl 使用方法

安装方法：https://docs.projectcalico.org/getting-started/calicoctl/install

我是在本地主机上安装的，下面是我的配置：

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

