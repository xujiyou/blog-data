# 配置 Pod 初始化

文档地址：https://kubernetes.io/zh/docs/tasks/configure-pod-container/configure-pod-initialization/

本文介绍在应用容器运行前，怎样利用 Init 容器初始化 Pod。

本例中您将创建一个包含一个应用容器和一个 Init 容器的 Pod。Init 容器在应用容器启动前运行完成。

下面是 Pod 的配置文件：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-demo
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 80
      volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: workdir
  initContainers:
    - name: install
      image: busybox
      command:
        - wget
        - "-O"
        - "/work-dir/index.html"
        - http://kubernetes.io
      volumeMounts:
        - mountPath: "/work-dir"
          name: workdir
  dnsPolicy: Default
  volumes:
    - name: workdir
      emptyDir: {}
```

这个 POD 非常有意思，两个容器公用一个挂载卷，初始化容器先往卷里面写入文件，然后 nginx 容器再使用这个文件。

检查：

```bash
[admin@fueltank-1 ~]$ kubectl get pod init-demo -o wide
NAME        READY   STATUS    RESTARTS   AGE   IP           NODE         NOMINATED NODE   READINESS GATES
init-demo   1/1     Running   0          11m   10.42.4.33   fueltank-2   <none>           <none>
```

访问：

```bash
$ curl 10.42.4.33 | grep "Kubernetes is open source giving you the freedom to take advantage"
```

