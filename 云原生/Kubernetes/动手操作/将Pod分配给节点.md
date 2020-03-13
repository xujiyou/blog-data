# 将 Pod 分配给节点

文档地址：https://kubernetes.io/zh/docs/tasks/configure-pod-container/assign-pods-nodes/

```bash
$ kubectl get nodes
$ kubectl label nodes fueltank-5 disktype=ssd
$ kubectl get nodes --show-labels
```

然后在创建 POD 时加上 nodeSelector ：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:
    disktype: ssd
```

看看 POD 会调度到 fueltank-5 上：

```bash
$ kubectl get pods --output=wide
NAME    READY   STATUS    RESTARTS   AGE   IP           NODE         NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          55s   10.42.2.22   fueltank-5   <none>           <none>
```