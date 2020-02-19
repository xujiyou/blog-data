# 使用 Secret 安全地分发凭证

本文展示如何安全地将敏感数据（如密码和加密密钥）注入到 Pods 中。

文档地址：https://kubernetes.io/zh/docs/tasks/inject-data-application/distribute-credentials-secure/

假设用户想要有两条 secret 数据：用户名 `my-app` 和密码 `39528$vdg7Jb`。 首先使用 [Base64 编码](https://www.base64encode.org/) 将用户名和密码转化为 base-64 形式。 这里是一个 Linux 示例：

```
echo -n 'my-app' | base64
echo -n '39528$vdg7Jb' | base64
```

结果显示 base-64 形式的用户名为 `bXktYXBw`， base-64 形式的密码为 `Mzk1MjgkdmRnN0pi`。

## 创建 Secret

这里是一个配置文件，可以用来创建存有用户名和密码的 Secret:

[`secret.yaml`](https://raw.githubusercontent.com/kubernetes/website/master/content/zh/examples/pods/inject/secret.yaml)

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: test-secret
data:
  username: bXktYXBwCg==
  password: Mzk1MjgkdmRnN0piCg==
```

```
kubectl get secret test-secret
kubectl describe secret test-secret
```

[`secret-pod.yaml`](https://raw.githubusercontent.com/kubernetes/website/master/content/zh/examples/pods/inject/secret-pod.yaml)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-test-pod
spec:
  containers:
    - name: test-container
      image: nginx
      volumeMounts:
          # name must match the volume name below
          - name: secret-volume
            mountPath: /etc/secret-volume
  # The secret data is exposed to Containers in the Pod through a Volume.
  volumes:
    - name: secret-volume
      secret:
        secretName: test-secret
```

```
kubectl create -f secret-pod.yaml
kubectl get pod secret-test-pod
kubectl exec -it secret-test-pod -- /bin/bash
cd /etc/secret-volume
ls
cat username; echo; cat password; echo
```

[`secret-envars-pod.yaml`](https://raw.githubusercontent.com/kubernetes/website/master/content/zh/examples/pods/inject/secret-envars-pod.yaml)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-envars-test-pod
spec:
  containers:
  - name: envars-test-container
    image: nginx
    env:
    - name: SECRET_USERNAME
      valueFrom:
        secretKeyRef:
          name: test-secret
          key: username
    - name: SECRET_PASSWORD
      valueFrom:
        secretKeyRef:
          name: test-secret
          key: password
```

```
kubectl get pod secret-envars-test-pod
kubectl exec -it secret-envars-test-pod -- /bin/bash
printenv
```

