# Jaeger-Operator

官方文档：https://www.jaegertracing.io/docs/1.17/operator/

Jaeger Operator 是 Kubernetes Operator的一个实现。

关于 Kubernetes Operator 可以参考： [Kubernetes-Operator教程.md](../Kubernetes/Kubernetes-Operator教程.md) 

## 在 Kubernetes 上安装 Jaeger Operator

```bash
$ kubectl create namespace observability # <1>
$ kubectl create -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/crds/jaegertracing.io_jaegers_crd.yaml # <2>
$ kubectl create -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/service_account.yaml
$ kubectl create -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/role.yaml
$ kubectl create -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/role_binding.yaml
$ kubectl create -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/operator.yaml
```

查看 刚刚部署的 operator：

```bash
$ kubectl get deployment jaeger-operator -n observability
```

