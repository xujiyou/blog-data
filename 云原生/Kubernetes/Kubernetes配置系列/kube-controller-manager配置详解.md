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

所有的控制器：

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





















