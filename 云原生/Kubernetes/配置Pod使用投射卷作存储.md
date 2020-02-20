# 配置 Pod 使用投射卷作存储

文档：https://kubernetes.io/zh/docs/tasks/configure-pod-container/configure-projected-volume-storage/

本文介绍怎样通过[`投射`](https://kubernetes.io/docs/concepts/storage/volumes/#projected) 卷将现有的多个卷资源挂载到相同的目录。 当前，`secret`、`configMap`、`downwardAPI` 和 `serviceAccountToken` 卷可以被投射。

[`projected.yaml`](https://raw.githubusercontent.com/kubernetes/website/master/content/zh/examples/pods/storage/projected.yaml)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-projected-volume
spec:
  containers:
  - name: test-projected-volume
    image: busybox
    args:
    - sleep
    - "86400"
    volumeMounts:
    - name: all-in-one
      mountPath: "/projected-volume"
      readOnly: true
  volumes:
  - name: all-in-one
    projected:
      sources:
      - secret:
          name: user
      - secret:
          name: pass
```

创建 Secrets:

```shell
<!--# Create files containing the username and password:--># 创建包含用户名和密码的文件:
   echo -n "admin" > ./username.txt
   echo -n "1f2d1e2e67df" > ./password.txt-->

<!--# Package these files into secrets:--># 将上述文件引用到 Secret：
   kubectl create secret generic user --from-file=./username.txt
   kubectl create secret generic pass --from-file=./password.txt
```

创建 Pod：

```shell
kubectl create -f https://k8s.io/examples/pods/storage/projected.yaml
```

确认 Pod 中的容器运行正常，然后监视 Pod 的变化：

```shell
kubectl get --watch pod test-projected-volume
```

在另外一个终端中，打开容器的 shell：

```shell
kubectl exec -it test-projected-volume -- /bin/sh
```

在 shell 中，确认 `projected-volume` 目录包含你的投射源：

```shell
ls /projected-volume/
```