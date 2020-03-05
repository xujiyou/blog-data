# Kubernetes Ingress Controller

为保证 Ingress 能正常工作，需要先安装 Ingress Controller。

这里我选 Nginx Ingress Controller

安装文档：https://kubernetes.github.io/ingress-nginx/deploy/

使用 Helm 安装：

```bash
$ kubectl create ns ingress
$ helm repo add stable https://kubernetes-charts.storage.googleapis.com/
$ helm install my-nginx stable/nginx-ingress --set rbac.create=true -n ingress
```

搞定。