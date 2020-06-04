# 批量删除Evicted状态的pod

由于node节点资源不足，造成资源的争抢，并出现大量的驱逐的pod，可以使用grep Evicted查看哪些pod

```bash
$ kubectl get pods -n istio-system | grep Evicted 
```

批量删除：

```bash
$ kubectl get pods -n kube-system | grep Evicted | awk '{print $1}' | xargs kubectl -n kube-system delete pod
```

排除第一行，批量删除：

```bash
$ kubectl get pods -n linli | awk '{if (NR>=2){print $1}}' | xargs kubectl delete pod -n linli
```

