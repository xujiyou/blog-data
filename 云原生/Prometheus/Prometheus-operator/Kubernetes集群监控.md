# Kubernetes 集群监控

集群监控分别包括几个方面：宿主机监控、kube-apiserver 监控、kubelet 监控；grafana、prometheus等组件自身的监控；自定义的监控。

Prometheus 默认以 pull 的方式获取指标，如果 pull 的方式行不通，可以考虑使用 Prometheus Gateway 采用 push 的方式。



## 监控宿主机

Prometheus 想要监控主机，需要在每个主机上部署 Node Exporter，在 Kubernetes 中，需要把这些 Node Exporter 封装成 Pod 部署在每一个节点之上。这可以用 DaemonSet 来实现，然后配合 HostPort。

Node Exporter会监听每个宿主机的 9796 端口。

可以直接通过 http://drift-1:9796/metrics 访问宿主机有哪些 metrics。

然后把这个端口封装在 Service 内，再创建一个 ServiceMonitor 实例即可实现对所有宿主机的监控。



## kube-apiserver 监控

Kube-apiserver 原生支持 Prometheus 的指标，可以通过以下方式查看 Metrics：

```bash
$ kubectl proxy &
$ curl http://127.0.0.1:8001/metrics
```



## kubelet 监控

kubelet 有个健康检查端口：

```bash
$ curl http://127.0.0.1:10248/healthz
```

kubelet 的 metrics 地址为：

```bash
$ curl -k https://127.0.0.1:10250/metrics
```

不过需要认证。















