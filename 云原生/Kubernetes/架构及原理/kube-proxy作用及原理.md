# kube-proxy 作用及原理



## Kubernetes 中的 proxy

用户在使用 Kubernetes 的过程中可能遇到几种不同的代理（proxy）：

1. [kubectl proxy](https://kubernetes.io/zh/docs/tasks/access-application-cluster/access-cluster/#directly-accessing-the-rest-api)：
   - 运行在用户的桌面或 pod 中
   - 从本机地址到 Kubernetes apiserver 的代理
   - 客户端到代理使用 HTTP 协议
   - 代理到 apiserver 使用 HTTPS 协议
   - 指向 apiserver
   - 添加认证头信息

2. [apiserver proxy](https://kubernetes.io/zh/docs/tasks/access-application-cluster/access-cluster/#discovering-builtin-services)：

- 是一个建立在 apiserver 内部的“堡垒”
- 将集群外部的用户与群集 IP 相连接，这些IP是无法通过其他方式访问的
- 运行在 apiserver 进程内
- 客户端到代理使用 HTTPS 协议 (如果配置 apiserver 使用 HTTP 协议，则使用 HTTP 协议)
- 通过可用信息进行选择，代理到目的地可能使用 HTTP 或 HTTPS 协议
- 可以用来访问 Node、 Pod 或 Service
- 当用来访问 Service 时，会进行负载均衡

3. [kube proxy](https://kubernetes.io/zh/docs/concepts/services-networking/service/#ips-and-vips)：

- 在每个节点上运行
- 代理 UDP、TCP 和 SCTP
- 不支持 HTTP
- 提供负载均衡能力
- 只用来访问 Service

4. apiserver 之前的代理/负载均衡器：

- 在不同集群中的存在形式和实现不同 (如 nginx)
- 位于所有客户端和一个或多个 API 服务器之间
- 存在多个 API 服务器时，扮演负载均衡器的角色

5. 外部服务的云负载均衡器：

- 由一些云供应商提供 (如 AWS ELB、Google Cloud Load Balancer)
- Kubernetes 服务类型为 `LoadBalancer` 时自动创建
- 通常仅支持 UDP/TCP 协议
- SCTP 支持取决于云供应商的负载均衡器实现
- 不同云供应商的云负载均衡器实现不同

Kubernetes 用户通常只需要关心前两种类型的代理，集群管理员通常需要确保后面几种类型的代理设置正确。





## kube-proxy

kube-proxy是Kubernetes的核心组件，部署在每个Node节点上，它是实现Kubernetes Service的通信与负载均衡机制的重要组件; kube-proxy负责为Pod创建代理服务，从apiserver获取所有Service信息，并根据Service信息创建代理服务，实现Service到Pod的请求路由和转发，从而实现K8s层级的虚拟转发网络。



## kube-proxy 代理模式

kube-proxy支持三种代理模式: 用户空间，iptables和IPVS；它们各自的操作略有不同。

#### Userspace

作为一个例子，考虑前面提到的图片处理应用程序。 当创建后端 Service 时，Kubernetes master 会给它指派一个虚拟 IP 地址，比如 10.0.0.1。 假设 Service 的端口是 1234，该 Service 会被集群中所有的 `kube-proxy` 实例观察到。 当代理看到一个新的 Service， 它会打开一个新的端口，建立一个从该 VIP 重定向到新端口的 iptables，并开始接收请求连接。

当一个客户端连接到一个 VIP，iptables 规则开始起作用，它会重定向该数据包到 "服务代理" 的端口。 "服务代理" 选择一个后端，并将客户端的流量代理到后端上。

这意味着 Service 的所有者能够选择任何他们想使用的端口，而不存在冲突的风险。 客户端可以简单地连接到一个 IP 和端口，而不需要知道实际访问了哪些 Pod。

#### iptables

再次考虑前面提到的图片处理应用程序。 当创建后端 Service 时，Kubernetes 控制面板会给它指派一个虚拟 IP 地址，比如 10.0.0.1。 假设 Service 的端口是 1234，该 Service 会被集群中所有的 `kube-proxy` 实例观察到。 当代理看到一个新的 Service， 它会配置一系列的 iptables 规则，从 VIP 重定向到每个 Service 规则。 该特定于服务的规则连接到特定于 Endpoint 的规则，而后者会重定向（目标地址转译）到后端。

当客户端连接到一个 VIP，iptables 规则开始起作用。一个后端会被选择（或者根据会话亲和性，或者随机）， 数据包被重定向到这个后端。 不像用户空间代理，数据包从来不拷贝到用户空间，kube-proxy 不是必须为该 VIP 工作而运行， 并且客户端 IP 是不可更改的。 当流量打到 Node 的端口上，或通过负载均衡器，会执行相同的基本流程， 但是在那些案例中客户端 IP 是可以更改的。

#### IPVS

在大规模集群（例如 10000 个服务）中，iptables 操作会显着降低速度。 IPVS 专为负载平衡而设计，并基于内核内哈希表。 因此，您可以通过基于 IPVS 的 kube-proxy 在大量服务中实现性能一致性。 同时，基于 IPVS 的 kube-proxy 具有更复杂的负载平衡算法（最小连接，局部性，加权，持久性）。

























