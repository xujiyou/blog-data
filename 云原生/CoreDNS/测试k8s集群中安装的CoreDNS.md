# 测试 k8s 集群中安装的 CoreDNS

首先安装 busybox：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox1
  labels:
    app: busybox1
spec:
  containers:
  - image: busybox:1.28.4
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
    name: busybox
  restartPolicy: Always
```

注意要使用 <= 1.28.4 版本的 busybox ，否则会有错误。

查看pods：

```bash
$ kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
busybox1                          1/1     Running   0          21s
```

随便找个 service：

```bash
[admin@fueltank-1 ~]$ kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
hello        ClusterIP   10.43.130.214   <none>        8080/TCP   3d15h
kubernetes   ClusterIP   10.43.0.1       <none>        443/TCP    19d
```

其他命名空间的 service：

```bash
[admin@fueltank-1 ~]$ kubectl get svc -n rook-ceph
NAME                       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
rook-ceph-mgr-dashboard    NodePort    10.43.203.169   <none>        8443:31912/TCP      11d
```

下面来查找上边这仨 service 的 ip 地址。

进入 busybox Pod：

```bash
[admin@fueltank-1 ~]$ kubectl exec busybox1 -i -t /bin/sh
/ # nslookup kubernetes.default
Server:    10.43.0.10
Address 1: 10.43.0.10 kube-dns.kube-system.svc.cluster.local

Name:      kubernetes.default
Address 1: 10.43.0.1 kubernetes.default.svc.cluster.local
/ # nslookup hello
Server:    10.43.0.10
Address 1: 10.43.0.10 kube-dns.kube-system.svc.cluster.local

Name:      hello
Address 1: 10.43.130.214 hello.default.svc.cluster.local
/ # nslookup hello.default
Server:    10.43.0.10
Address 1: 10.43.0.10 kube-dns.kube-system.svc.cluster.local

Name:      hello.default
Address 1: 10.43.130.214 hello.default.svc.cluster.local
/ # nslookup rook-ceph-mgr-dashboard.rook-ceph
Server:    10.43.0.10
Address 1: 10.43.0.10 kube-dns.kube-system.svc.cluster.local

Name:      rook-ceph-mgr-dashboard.rook-ceph
Address 1: 10.43.203.169 rook-ceph-mgr-dashboard.rook-ceph.svc.cluster.local
```

可以看到使用 nslookup 查询出来的 service 地址和 `kubectl get svc` 得到的 CLUSTER-IP 是一模一样的，证明工作完美。

查看集群中 CoreDNS 的服务地址：

```bash
$ kubectl get svc -n kube-system
NAME             TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                  AGE
kube-dns         ClusterIP   10.43.0.10     <none>        53/UDP,53/TCP,9153/TCP   20d
```

可以看到，和上边的服务地址是一样的。