# kube-controller-manager 配置详解

官方文档地址：https://kubernetes.io/zh/docs/reference/command-line-tools-reference/kube-controller-manager/

共120多个配置项



## Debugging flags

#### --contention-profiling

如果启用了概要分析，则启用锁争用概要分析

#### --profiling

Enable profiling via web interface host:port/debug/pprof/



## Generic flags

#### --allocate-node-cidrs

【Cloud】云服务商提供的 Pod 的地址池。



#### --cidr-allocator-type string

CIDR 类型，默认 RangeAllocator



#### --cloud-config string 

【Cloud】云提供程序配置文件的路径。 空字符串意味着无配置文件。



#### --cloud-provider string

【Cloud】云服务商，为空则意味着没有



#### --cluster-cidr string

Pod 的地址池，需要 --allocate-node-cidrs 为 true。



#### --cluster-name string

集群实例名称的前缀，默认为 kubernetes



#### --configure-cloud-routes

【Cloud】是否应在云提供商上配置由allocate-node-cidrs分配的CIDR。默认为 true



#### --controller-start-interval duration 

启动控制器管理器之间的间隔。



#### --controllers strings

开启控制器的列表，* 意味着启用所有默认控制器，“foo” 表示开启名字为 foo 的控制器，“-foo” 表示关闭名字为 foo 的控制器。

所有默认开启的控制器，共36个：

attachdetach, bootstrapsigner, cloud-node-lifecycle, clusterrole-aggregation, cronjob, csrapproving, csrcleaner, csrsigning, daemonset, deployment, disruption, endpoint, endpointslice, garbagecollector, horizontalpodautoscaling, job, namespace, nodeipam, nodelifecycle, persistentvolume-binder, persistentvolume-expander, podgc, pv-protection, pvc-protection, replicaset, replicationcontroller, resourcequota, root-ca-cert-publisher, route, service, serviceaccount, serviceaccount-token, statefulset, tokencleaner, ttl, ttl-after-finished。

默认关闭的控制器：

 bootstrapsigner, tokencleaner

默认值为 [*]



#### --external-cloud-volume-plugin string

【Cloud】将云提供商设置为外部时使用的插件。 可以为空，仅应在外部云提供程序时设置。 当前用于允许节点和卷控制器在树云提供程序中工作。



#### --feature-gates mapStringBool

特性开关



#### --kube-api-burst int32

与kubernetes apiserver聊天时可使用。默认 30.



#### --kube-api-content-type string

与 apiserver 交互时使用的数据类型，默认为 application/vnd.kubernetes.protobuf



#### --kube-api-qps float32 

与 apiserver 交互时使用的 QPS，默认 20



#### --leader-elect 

是否进行 leader 选举，可以保证高可用，默认为 true



#### --leader-elect-lease-duration duration 

leader 选举的租约时间，默认 15s



#### --leader-elect-renew-deadline duration

也是关于 leader 选举的时间，默认 10s，这个时间必须小于等于  --leader-elect-lease-duration 



#### --leader-elect-resource-lock endpoints

leader 选举时使用的 资源对象类型锁，默认为 “endpointsleases”，支持的选项为 endpoints （默认）和 “configmaps”



#### --leader-elect-resource-name string 

leader 选举时使用的 资源对象类型名字，默认为 kube-controller-manager



#### --leader-elect-resource-namespace string

leader 选举时使用的 资源对象类型命名空间，默认为 kube-system



#### --leader-elect-retry-period duration 

leader 选举重试间隔，默认 2s



#### --min-resync-period duration

反射器中的重新同步周期在MinResyncPeriod和2 * MinResyncPeriod之间是随机的，默认 12h



#### --node-monitor-period duration

在 NodeController 中，同步 NodeStatus 的时间



#### --route-reconciliation-period duration

【Cloud】云提供商为节点创建的路由协调时间，默认 10s



#### --use-service-account-credentials 

如果为true，则为每个控制器使用单独的 ServiceAccount 凭据。



## Service controller flags

#### --concurrent-service-syncs int32

允许同时同步的服务数，数量与服务管理响应速度有关，数量更大=服务管理响应速度更快，但CPU（和网络）负载更多，默认为 1



## Secure serving flags

安全选项

#### --bind-address ip

监听的 IP 地址，默认为 0.0.0.0



#### --cert-dir string 

证书目录



#### --http2-max-streams-per-connection int

HTTP 流的连接数量限制，0意味着使用 golang 默认的。



#### --secure-port int 

监听的端口，默认 10257



#### --tls-cert-file string

证书文件



#### --tls-cipher-suites strings

加密方法列表，全部的加密方式包括：

TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA,TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA,TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_ECDSA_WITH_RC4_128_SHA,TLS_ECDHE_RSA_WITH_3DES_EDE_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_RC4_128_SHA,TLS_RSA_WITH_3DES_EDE_CBC_SHA,TLS_RSA_WITH_AES_128_CBC_SHA,TLS_RSA_WITH_AES_128_CBC_SHA256,TLS_RSA_WITH_AES_128_GCM_SHA256,TLS_RSA_WITH_AES_256_CBC_SHA,TLS_RSA_WITH_AES_256_GCM_SHA384,TLS_RSA_WITH_RC4_128_SHA



#### --tls-min-version string

TLS 最小版本，可选值为 VersionTLS10, VersionTLS11, VersionTLS12, VersionTLS13



#### --tls-private-key-file string

私钥文件



#### --tls-sni-cert-key namedCertKey

证书文件名，不包括后缀，比如"example.crt,example.key" or "foo.crt,foo.key:*.foo.com,foo.com". (default [])



## Authentication flags

#### --authentication-kubeconfig string 

kubeconfig 文件地址，如果为空，则所有令牌请求均被视为匿名请求，并且不会在群集中查找任何客户端CA。



#### --authentication-skip-lookup 

如果为false，则authentication-kubeconfig将用于从群集中查找缺少的身份验证配置。



#### --authentication-token-webhook-cache-ttl duration

The duration to cache responses from the webhook token authenticator. (default 10s)



#### --authentication-tolerate-lookup-failure

如果为true，则无法从群集中查找缺少的身份验证配置失败是致命的。 请注意，这可能导致身份验证将所有请求视为匿名。



#### --client-ca-file string

CA 文件



#### --requestheader-allowed-names strings

客户端证书通用名称列表，允许在--requestheader-username-headers指定的标头中提供用户名。 如果为空，则允许在--requestheader-client-ca-file中由授权机构验证的任何客户端证书。



#### --requestheader-client-ca-file string

在信任--requestheader-username-headers指定的标头中的用户名之前，用于验证传入请求上的客户端证书的根证书捆绑包。 警告：通常不依赖于传入请求已经完成的授权。



#### --requestheader-extra-headers-prefix strings

List of request header prefixes to inspect. X-Remote-Extra- is suggested. (default [x-remote-extra-])



#### --requestheader-group-headers strings

List of request headers to inspect for groups. X-Remote-Group is suggested. (default [x-remote-group])



#### --requestheader-username-headers strings 

List of request headers to inspect for usernames. X-Remote-User is common. (default [x-remote-user])





## Authorization flags

#### --authorization-always-allow-paths strings

跳过授权的 Path 列表，(default [/healthz])



#### --authorization-kubeconfig string

指向具有足够权限以创建subjectaccessreviews.authorization.k8s.io的“核心” kubernetes服务器的kubeconfig文件。 这是可选的。 如果为空，则禁止所有未经授权请求。



#### --authorization-webhook-cache-authorized-ttl duration 

 The duration to cache 'authorized' responses from the webhook authorizer. (default 10s)



#### --authorization-webhook-cache-unauthorized-ttl duration 

The duration to cache 'unauthorized' responses from the webhook authorizer. (default 10s)



## Attachdetach controller flags

Attachdetach controller 用于 操作 kubelet 使用到的 Volumn

#### --attach-detach-reconcile-sync-period duration

卷连接分离之间的协调器同步等待时间。 此持续时间必须大于一秒，并且将此值从默认值开始增加可能会导致卷与Pod不匹配。 （默认为1m0s）



#### --disable-attach-detach-reconcile-sync 

禁用卷附加分离协调程序同步。 禁用此功能可能导致卷与 Pod 不匹配。 谨慎使用。



## Csrsigning controller flags

Csrsigning controller用于管理签名

#### --cluster-signing-cert-file string

包含用于发布群集范围的证书的，包含PEM编码的X509 CA证书的文件名，默认为 /etc/kubernetes/ca/ca.pem



#### --cluster-signing-key-file string

包含用于对群集范围的证书进行签名的PEM编码的RSA或ECDSA私钥的文件名，默认为 /etc/kubernetes/ca/ca.key



#### --experimental-cluster-signing-duration duration

将给出签名证书的有效期限。默认 8760h，一年



## Deployment controller flags

#### --concurrent-deployment-syncs int32

允许同时同步的部署对象的数量。 更大的数量=更具响应性的部署，但更多的CPU（和网络）负载（默认5）



#### --deployment-controller-sync-period duration 

同步 deployment 的时间段。 （默认30秒）



## Statefulset controller flags

#### --concurrent-statefulset-syncs int32

允许同时同步的statefulset对象的数量。 较大的数量=响应状态集更多，但CPU（和网络）负载更多（默认5）



## Endpoint controller flags

#### --concurrent-endpoint-syncs int32

将同时完成的端点同步操作数。 更大的数量=更快的端点更新，但是更多的CPU（和网络）负载（默认5）



#### --endpoint-updates-batch-period duration

端点的长度将更新批处理周期。 Pod更改的处理将延迟此持续时间，以使它们与潜在的即将进行的更新结合在一起，并减少端点更新的总数。 较大的数量=较高的端点编程延迟，但是生成的端点修订版本数量较少



## Endpointslice controller flags

介绍：https://kubernetes.io/zh/docs/concepts/services-networking/endpoint-slices/

Endpoint Slices* 提供了一种简单的方法来跟踪 Kubernetes 集群中的网络端点（network endpoints）。它们为 Endpoints 提供了一种可伸缩和可拓展的替代方案。

#### --concurrent-service-endpoint-syncs int32

将同时完成的服务端点同步操作的数量。 更大的数量=更快的端点切片更新，但是更多的CPU（和网络）负载。
（默认为5）



#### --max-endpoints-per-slice int32

将添加到EndpointSlice的最大端点数。 每个切片的端点越多，端点切片将越少，但资源却越大。
（默认为100）



## Garbagecollector controller flags

查看介绍：https://kubernetes.io/zh/docs/concepts/workloads/controllers/garbage-collection/

Kubernetes 垃圾收集器的作用是删除某些曾经拥有所有者（owner）但现在不再拥有所有者的对象。

#### --concurrent-gc-syncs int32

允许同时同步的垃圾收集器工作程序的数量。 （默认为20）



#### --enable-garbage-collector

启用通用垃圾收集器。 必须与kube-apiserver的相应标志同步，默认为 true



## Horizontalpodautoscaling controller flags

#### --horizontal-pod-autoscaler-cpu-initialization-period duration

启动后 Pod 可能会跳过CPU采样的时间段。 （默认为5m0s）

#### 

#### --horizontal-pod-autoscaler-downscale-stabilization duration

自动定标器将向后看的时间段，并且不会缩小到该时间段内的建议值以下。 （默认为5m0s）



#### --horizontal-pod-autoscaler-initial-readiness-delay duration

Pod 启动后准备状态发生变化的时间段将被视为初始准备状态。 （默认30秒）



#### --horizontal-pod-autoscaler-sync-period duration 

在水平容器自动缩放器中同步容器数的时间段。 （默认15秒）



#### --horizontal-pod-autoscaler-tolerance float

水平 Pod 自动缩放器要考虑缩放比例时，期望值与实际度量值之比的最小变化（从1.0起）。 （默认为0.1）



## Namespace controller flags

#### --concurrent-namespace-syncs int32

允许同时同步的名称空间对象的数量。 较大的数字=响应性更好的名称空间终止，但是更多的CPU（和网络）负载，默认 10



#### --namespace-sync-period duration

同步名称空间生命周期更新的时间段（默认为5m0s）



## Nodeipam controller flags

#### --node-cidr-mask-size int32

群集中节点cidr的掩码大小。 IPv4的默认值为24，IPv6的默认值为64。



#### --node-cidr-mask-size-ipv4 int32

双栈群集中IPv4节点cidr的掩码大小。 默认值为24。



#### --node-cidr-mask-size-ipv6 int32

双栈群集中IPv6节点cidr的掩码大小。 默认值为64。



#### --service-cluster-ip-range string

群集中服务的CIDR范围。 要求--allocate-node-cidrs为true



## Nodelifecycle controller flags

#### --enable-taint-manager

警告：Beta功能。 如果设置为true，则启用NoExecute Taints并将逐出在受此Taints污染的Node上运行的所有非容忍Pod。 （默认为true）



#### --large-cluster-size-threshold int32

出于驱逐逻辑的目的，NodeController从中将群集视为较大的节点数。 对于此大小或更小的群集，--secondary-node-eviction-rate被隐式覆盖为0。 （默认为50）



#### --node-eviction-rate float32

如果某个区域处于健康状态，则在节点发生故障的情况下每秒删除Pod的节点数（请参阅--unhealthy-zone-threshold，以了解“健康” /“不健康”的定义）。 区域是指非多区域群集中的整个群集。 （默认为0.1）



#### --node-monitor-grace-period duration

在标记为运行状况不正常之前，我们允许运行的Node停止响应的时间。 必须是kubelet的nodeStatusUpdateFrequency的N倍，其中N表示允许kubelet发布节点状态的重试次数。 （默认40秒）



#### --node-startup-grace-period duration

在标记为不正常之前，我们允许启动Node停止响应的时间。 （默认为1m0s）



#### --pod-eviction-timeout duration

删除故障节点上的Pod的宽限期。 （默认为5m0s）



#### --secondary-node-eviction-rate float32

当区域不健康时，如果节点发生故障，每秒删除Pod的节点数（有关健康/不健康的定义，请参阅--unhealthy-zone-threshold）。 区域是指非多区域群集中的整个群集。 如果群集大小小于--large-cluster-size-threshold，则此值隐式覆盖为0。 （默认0.01）



#### --unhealthy-zone-threshold float32

区域中的节点分数不需要准备就绪（最低3），才能将其视为不健康区域。 （默认为0.55）



## Persistentvolume-binder controller flags

#### --enable-dynamic-provisioning

为支持它的环境启用动态预配置。 （默认为true）



#### --enable-hostpath-provisioner

在没有云提供商的情况下运行时，请启用HostPath PV置备。 这允许测试和开发配置功能。 根本不支持HostPath置备，它不能在多节点群集中工作，并且除测试或开发外，不得用于其他任何用途。



#### --flex-volume-plugin-dir string

Flex卷插件应在其中搜索其他第三方卷插件的目录的完整路径。

默认：/usr/libexec/kubernetes/kubelet-plugins/volume/exec/



#### --pv-recycler-increment-timeout-nfs int32

NFS洗涤器容器的每个Gi添加到ActiveDeadlineSeconds的时间增量（默认为30）



#### --pv-recycler-minimum-timeout-hostpath int32

用于HostPath Recycler窗格的最小ActiveDeadlineSeconds。 这仅用于开发和测试，不适用于多节点群集。 （默认为60）



#### --pv-recycler-minimum-timeout-nfs int32 

NFS回收站窗格使用的最小ActiveDeadlineSeconds（默认为300）



#### --pv-recycler-pod-template-filepath-hostpath string

Pod定义的文件路径，用作HostPath持久卷回收的模板。 这仅用于开发和测试，不适用于多节点群集。



#### --pv-recycler-pod-template-filepath-nfs string

Pod定义的文件路径，用作NFS持久卷回收的模板



#### --pv-recycler-timeout-increment-hostpath int32 

HostPath洗涤器容器的每个Gi添加到ActiveDeadlineSeconds的时间增量。 这仅用于开发和测试，不适用于多节点群集。 （默认为30）



#### --pvclaimbinder-sync-period duration 

同步永久卷和永久卷声明的时间段（默认为15秒）



## Podgc controller flags

#### --terminated-pod-gc-threshold int32

在终止的Pod垃圾收集器开始删除终止的Pod之前可以存在的终止的Pod数。 如果<= 0，则禁用终止的pod垃圾收集器。 （默认为12500）



## Replicaset controller flags

#### --concurrent-replicaset-syncs int32

允许同时同步的副本集的数量。 较大的数量=副本管理响应速度更快，但CPU（和网络）负载更多（默认5）



## Replicationcontroller flags

#### --concurrent_rc_syncs int32

允许同时同步的复制控制器的数量。 数量更多=副本管理响应速度更快，但CPU（和网络）负载更多
（默认为5）



## Resourcequota controller flags

#### --concurrent-resource-quota-syncs int32

允许同时同步的资源配额数。 更大的数量=响应性更强的配额管理，但更多的CPU（和网络）负载（默认5）



#### --resource-quota-sync-period duration

系统中配额使用状态的同步时间（默认值为5m0s）



## Serviceaccount controller flags

#### --concurrent-serviceaccount-token-syncs int32

允许同时同步的服务帐户令牌对象的数量。 较大的数字=响应令牌生成速度更快，但CPU（和网络）负载更多（默认5）



#### --root-ca-file string

如果设置，则此根证书颁发机构将包含在服务帐户的令牌密钥中。 这必须是有效的PEM编码的CA捆绑包。



#### --service-account-private-key-file string

包含用于对服务帐户令牌进行签名的PEM编码的私有RSA或ECDSA密钥的文件名。





## Ttl-after-finished controller flags

#### --concurrent-ttl-after-finished-syncs int32 

允许同时同步的TTL后完成的控制器工作程序的数量。 （默认为5）



## Misc flags

#### --kubeconfig string

具有授权和主位置信息的kubeconfig文件的路径。



#### --master string

Kubernetes API服务器的地址（覆盖kubeconfig中的任何值）。



## Global flags

#### --add-dir-header

 If true, adds the file directory to the header



#### --alsologtostderr

log to standard error as well as files



#### -h, --help

查看帮助



#### --log-backtrace-at traceLocation

when logging hits line file:N, emit a stack trace (default :0)



#### --log-dir string

If non-empty, write log files in this directory



#### --log-file string

If non-empty, use this log file



#### --log-file-max-size uint

Defines the maximum size a log file can grow to. Unit is megabytes. If the value is 0, the maximum file size is unlimited. (default 1800)



#### --log-flush-frequency duration

Maximum number of seconds between log flushes (default 5s)



#### --logtostderr 

log to standard error instead of files (default true)



#### --skip-headers 

If true, avoid header prefixes in the log messages



#### --skip-log-headers

If true, avoid headers when opening log files



####  --stderrthreshold severity

logs at or above this threshold go to stderr (default 2)



#### -v, --v Level

number for the log level verbosity



#### --version version[=true]

Print version information and quit



#### --vmodule moduleSpec

comma-separated list of pattern=N settings for file-filtered logging









