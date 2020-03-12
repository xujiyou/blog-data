# kube-apiserver 配置详解

kube-apiserver 配置官方文档：https://kubernetes.io/zh/docs/reference/command-line-tools-reference/kube-apiserver/

官方文档的配置是没经过分类的。可以通过以下命令来查看分类过后的全部配置选项：

```bash
$ kube-apiserver -h
```

在源码中，kubernetes 的组件的命令行参数都是通过 cobra 来构建的。下面讲解时也会讲配置在源码中的位置。

首先，kube-apiserver 的配置是在 `cmd/kube-apiserver/app/server.go` 中初始化的。

然后，kube-apiserver 的配置分类在 `cmd/kube-apiserver/app/options/options.go` 中。

下面一组一组的学习。我使用的 k8s 版本是 1.17.3

## Generic flags

通用配置的定义全在 `staging/src/k8s.io/apiserver/pkg/server/options/server_run_options.go` 中。

一共有十六个配置，其中有一个特性开关配置。

- **--advertise-address ip** 将apiserver通告给集群成员的IP地址。其余集群必须可以访问此地址。 如果为空，将使用--bind-address。 如果未指定--bind-address，则将使用主机的默认接口。（一般使用默认即可）
- **--cloud-provider-gce-l7lb-src-cidrs cidrs** GCE 的配置，（不用管）
- **--cors-allowed-origins strings **CORS允许的来源清单，以逗号分隔。 允许的来源可以是支持子域匹配的正则表达式。 如果此列表为空，则不会启用CORS。（跨域协议，如果想安全可以设置）
- **--default-not-ready-toleration-seconds int** 等待notReady:NotExecute的秒数,默认300,默认会给所有未设置的toleration的pod设置该值，节点 NotReady 超过这个时间容器就被干掉。（默认即可）
- **--default-unreachable-toleration-seconds int** 同上，unreachable:NoExecute 的容忍秒数，默认情况下添加到尚未具有此容忍度的每个Pod中。 （默认为300）（默认即可）
- **--enable-inflight-quota-handler** 在公平调度和优先调度之间切换 （一般不设置即可）
- **--external-hostname string ** 外部访问 kube-apiserver 时使用的主机名 （设置成本机主机名即可）
- **--livez-grace-period duration** 此选项代表apiserver完成启动并生效所需的最长时间。 从apiserver的启动时间到这段时间为止，这段时间内 /livez 将返回 true （如果能把握好启动时间就设置上）
- **--master-service-namespace string ** DEPRECATED: 过期了，如果不指定 namespace 的话，使用的默认 namespace，默认为 “default” （不要设置）
- **--max-mutating-requests-inflight int** 指定时间内的最大的修改请求数量，如果超过了了这个值，kube-apiserver 会拒绝请求。默认为 200，0表示无限制。（对k8s熟练掌握了，对当前集群了解的话设置）
- **--max-requests-inflight int** 同上，不过请求是非破坏的，默认是 400（对k8s熟练掌握了，对当前集群了解的话设置）
- **--min-request-timeout int** 可选字段，暂时理解为 watch 请求的超时时间，默认值 1800 （可不设置）
- **--request-timeout duration** 可选字段，请求超时时间，这是默认请求超时，对于特定的请求可能会被 --min-request-timeout 覆盖，默认值为1分钟。（可不设置）
- **--shutdown-delay-duration duration** 终止时间设置，在这段时间内，继续处理请求，但不会再接受请求了，相当于黄灯时间，/healthz 返回成功，但是 /readyz 立即返回失败。这可用于允许负载平衡器停止向该服务器发送流量。
- **--target-ram-mb int** 内存限制，以MB为单位，用于配置缓存大小等。

另外，特性开关最后学习。



## Etcd flags

etch falgs 的源码在 `staging/src/k8s.io/apiserver/pkg/server/options/etcd.go`

一个 16 个配置项。

- **--default-watch-cache-size int** 默认 watch 资源的缓存大小，如果为零，将对未设置默认监视大小的资源禁用监视缓存。（默认值100）
- **--delete-collection-workers** 删除工作的协程数量，这些可用于加速命名空间的清理，默认为1
- **--enable-garbage-collector** 启用通用垃圾收集器。 必须与kube-controller-manager的相应的配置同步。 （默认为true）
- **--encryption-provider-config string** 一个文件路径，这个文件提供了配置，配置如何加密保存 etcd 中的 secrets。
- **--etcd-cafile** etcd 连接用的 CA
- **--etcd-certfile string ** etcd 连接用的证书
- **--etcd-compaction-interval duration** 压缩请求的间隔。 如果为0，则禁用来自apiserver的压缩请求。 （默认为5m0s）
- **--etcd-count-metric-poll-period duration** 针对每种类型的资源数量轮询etcd的频率。 0禁用度量标准收集。 （默认为1m0s）
- **--etcd-keyfile string** etcd 的私钥
- **--etcd-prefix string** etcd 键的前缀，默认是 "/registry"
- **--etcd-servers strings** etcd 的连接地址，可多个，逗号分隔
- **--etcd-servers-overrides strings** 每个资源的etcd服务器会覆盖，以逗号分隔。 单个替代格式：group/resource#servers，其中服务器是URL，以分号分隔。
- **--storage-backend string** 储存后端，默认是 etcd3
- **--storage-media-type string** etcd 中储存对象的类型，默认是  "application/vnd.kubernetes.protobuf"
- **--watch-cache** 开启 watch 缓存，默认为 true
- **--watch-cache-sizes strings** watch 缓存大小，可设置多个资源，以逗号分隔。格式为 resource[.group]#size ，没有设置的将使用 --default-watch-cache-size 配置的值。



## Secure serving flags

安全服务，9 个配置源码在：`staging/src/k8s.io/apiserver/pkg/server/options/serving.go`

- **--bind-address ip** 绑定的 IP 地址，访问 kube-apiserver 时使用的 IP 地址，如果为空，则为 0.0.0.0，允许所有地址访问。
- **--cert-dir string** 存放证书及私钥文件的目录地址，如果--tls-cert-file 和 --tls-private-key-file 配置设置了，这个配置将被忽略，默认是在 "/var/run/kubernetes"
- **--http2-max-streams-per-connection int** 服务器为客户端提供的HTTP / 2连接中最大流数的限制。 零表示使用golang的默认值。
- **--secure-port int** 安全端口，默认 6443
- **--tls-cert-file string** 证书文件地址，如果 HTTPS 启用了，但是没有配置 --tls-cert-file 和 --tls-private-key-file，但是配置了 --cert-dir ，apiserver 将会自动生成证书到 --cert-dir 目录。
- **--tls-cipher-suites strings ** 服务器的密码套件列表，以逗号分隔。 如果省略，将使用默认的Go密码套件。 可能的值：TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA,TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA,TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_ECDSA_WITH_RC4_128_SHA,TLS_ECDHE_RSA_WITH_3DES_EDE_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA_WITH_RC4_128_SHA,TLS_RSA_WITH_3DES_EDE_CBC_SHA,TLS_RSA_WITH_AES_128_CBC_SHA,TLS_RSA_WITH_AES_128_CBC_SHA256,TLS_RSA_WITH_AES_128_GCM_SHA256,TLS_RSA_WITH_AES_256_CBC_SHA,TLS_RSA_WITH_AES_256_GCM_SHA384,TLS_RSA_WITH_RC4_128_SHA
- **--tls-min-version string** TLS 最小版本，可能的值：VersionTLS10, VersionTLS11, VersionTLS12, VersionTLS13
- **--tls-private-key-file string** 私钥地址
- **--tls-sni-cert-key namedCertKey** 允许访问的地址，可以访问到的地址，如果未提供，将会使用证书中的地址，例子："example.crt,example.key" 或者 "foo.crt,foo.key:*.foo.com,foo.com". (默认 []) 这样就不必每次都改证书中的合法地址了。



## Insecure serving flags

不安全的服务，4个配置，都过期了，不要用了，简单列一下：--address ，--insecure-bind-address，-insecure-port，--port



## Auditing flags

审计配置，31个配置，全部以 audit 开头，源码在：`staging/src/k8s.io/apiserver/pkg/server/options/audit.go`

- **--audit-dynamic-configuration** 





