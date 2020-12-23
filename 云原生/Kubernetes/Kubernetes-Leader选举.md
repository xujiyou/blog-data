# Kubernetes Leader选举

在上篇文章中： [Kubernetes二进制安装.md](Kubernetes二进制安装.md) 已经使用二进制方式部署了各个组件，但是感觉只有 etcd 使用了集群配置，其他的组件都没有集群配置，那他们是怎么实现高可用的那。

我们都知道 k8s 核心组件，其中 apiserver 只用于接收 api 请求，不会主动进行各种动作，所以他们在每个节点都运行并且都可以接收请求，不会造成异常；kube-proxy 也是一样，只用于做端口转发，不会主动进行动作执行。

apiserver 有 etcd 保证并发。

但是 scheduler, controller-manager 不同，他们参与了 Pod 的调度及具体的各种资源的管控，如果同时有多个 controller-manager 来对 Pod 资源进行调度，那么 k8s 是如何做到正确运转的呢？

k8s 所有功能都是通过 `services` 对外暴露接口，而 `services` 对应的是具体的 `endpoints` ，那么来看下 scheduler 和 controller-manager 的 `endpoints` 是什么：

```bash
$ kubectl -n kube-system get endpoints
NAME                      ENDPOINTS                                                      AGE
kube-controller-manager   <none>                                                         3d17h
kube-dns                  10.42.222.129:53,10.42.222.135:53,10.42.56.65:53 + 6 more...   3d17h
kube-scheduler            <none>                                                         3d17h
```

上安装过程中，kube-controller-manager 和 kube-scheduler 都配置了：--leader-elect=true

查看选举日志：

```bash
$ kubectl -n kube-system describe endpoints kube-scheduler
```



彩蛋：

如果想本地调试 kube-controller-manager 和 kube-scheduler ，只需把源码下载下来，用 Goland 打开，配置好 --kubeconfig 参数，启动 main 函数就可以了。



另外：

scheduler, controller-manager 能自己保证高可用，kubelet 和 kube-proxy 是运行在每个机器上的，单个机器挂了也不会出问题。剩下的只有 kube-apiserver 的高可用了。

在 k8s 集群内部，kube-apiserver 有负载均衡能保证高可用。

官方并没有给出外部调用 kube-apiserver 的高可用解决方案，只是依赖 etcd 实现了并发安全。

如何实现 kube-apiserver  的高可用那，可以使用外部负载均衡，也可以使用 keepalived 实现内部的负载均衡，也可以使用 nginx 做反向代理。



