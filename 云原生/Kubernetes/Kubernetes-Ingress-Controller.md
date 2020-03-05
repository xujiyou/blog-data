# Kubernetes Ingress Controller

为保证 Ingress 能正常工作，需要先安装 Ingress Controller。

这里我选 Nginx Ingress Controller

由于这个东西需要负载均衡，但是本地或私有云环境是没有负载均衡的，可以使用 Metallb

安装文档：https://metallb.universe.tf/installation/

```bash
$ kubectl apply -f https://raw.githubusercontent.com/google/metallb/v0.8.3/manifests/metallb.yaml
```

等待部署好之后，还需要创建 ConfigMap 提供 IP 池：

```bash
$ wget https://raw.githubusercontent.com/google/metallb/v0.8.3/manifests/example-layer2-config.yaml
```

下载下来之后，修改其中的 IP 地址池跟自己本地相近，然后部署：

```bash
$ kubectl apply -f example-layer2-config.yaml
```



然后安装 Nginx Ingress Controller

安装文档：https://kubernetes.github.io/ingress-nginx/deploy/

使用 Helm 安装：

```bash
$ helm repo add stable https://kubernetes-charts.storage.googleapis.com/
$ helm install my-nginx stable/nginx-ingress --set rbac.create=true -n kube-system
```

还需注意，国内无法下载其中的一个镜像，但是可以这样：

```bash
$ docker pull googlecontainer/defaultbackend-amd64:1.5
$ docker tag googlecontainer/defaultbackend-amd64:1.5 k8s.gcr.io/defaultbackend-amd64:1.5
```



查看效果：

![image-20200305132855703](../../resource/image-20200305132855703.png)

