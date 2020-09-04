# Pular 在 Kubernetes 中安装

在 https://github.com/apache/pulsar/releases 中下载最新的源码包。解压，进入目录，再执行：

```bash
$ cd deployment/kubernetes/helm/
$ ./scripts/pulsar/prepare_helm_release.sh -n pulsar -k pulsar-mini --control-center-admin pulsar  --control-center-password pulsar -c
$ helm install --values examples/values-local-cluster.yaml pulsar pulsar -n pulsar
```

直到所有 Pod 处于运行状态：

```bash
$ kubectl get pods -n pulsar
```



