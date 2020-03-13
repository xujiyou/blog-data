# kube-apiserver 配置详解

kube-apiserver 配置官方文档：https://kubernetes.io/zh/docs/reference/command-line-tools-reference/kube-apiserver/

共 90 多个配置项。

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

- **--audit-dynamic-configuration** 启用动态审计配置。 此功能还需要DynamicAuditing功能标志
- **--audit-log-batch-buffer-size int** 批处理和写入之前用于存储事件的缓冲区大小。 仅在批处理模式下使用。 （默认为10000）
- **--audit-log-batch-max-size int** 最大的批处理数量，默认为 1
- **--audit-log-batch-max-wait duration** 强制写入尚未达到最大大小的批处理之前要等待的时间。 仅在批处理模式下使用。
- **--audit-log-batch-throttle-burst int** 如果之前未使用ThrottleQPS，则同时发送的最大请求数。 仅在批处理模式下使用。
- **--audit-log-batch-throttle-enable** 是否启用了批量调节。 仅在批处理模式下使用。
- **--audit-log-batch-throttle-qps float32** 每秒的最大平均批数。 仅在批处理模式下使用。
- **--audit-log-format string** 审计的日志格式。 “legacy”表示每个事件的1行文本格式。 “ json”表示结构化的json格式。 已知格式为legacy，json。 （默认为“ json”）
- **--audit-log-maxage int** 根据文件名中编码的时间戳记保留旧审核日志文件的最大天数。
- **--audit-log-maxbackup int** 保留的旧审计日志文件的最大数量。
- **--audit-log-maxsize int** 保存之前，审核日志文件的最大大小（以兆字节为单位）。
- **--audit-log-mode string**  发送审核事件的策略。 Blocking 表示发送事件应阻止服务器响应。 批处理导致后端异步缓冲和写入事件。 已知的模式是batch,blocking,blocking-strict。 （默认为“blocking”）
- **--audit-log-path string** 审计日志保存地址，“-”表示标准输出。
- **--audit-log-truncate-enabled** 是否启用事件和批处理截断。
- **--audit-log-truncate-max-batch-size int ** 发送到基础后端的批处理的最大大小。 实际的序列化大小可能会增加数百个字节。 如果一个批次超出此限制，则将其分成几个较小的批次。 （默认10485760）
- **--audit-log-truncate-max-event-size int** 发送到基础后端的审核事件的最大大小。 如果事件的大小大于此数字，则将删除第一个请求和响应，并且如果事件的大小没有减小到足够的数量，则将事件丢弃。 （默认为102400）
- **--audit-log-version string** 用于序列化写入日志的审核事件的API组和版本。 （默认为“ audit.k8s.io/v1”）
- **--audit-policy-file string** 定义审核策略配置的文件的路径。
- **--audit-webhook-batch-buffer-size int** 批处理和写入之前用于存储事件的缓冲区大小。 仅在批处理模式下使用。 （默认为10000）
- **--audit-webhook-batch-max-size int ** 批处理的最大大小。 仅在批处理模式下使用。 （默认为400）

webhook 与 log 的区别就是发往文件还是网络，其他都是一样的。



## Features flags

- **--contention-profiling** 如果启用了概要分析，则启用锁争用概要分析
- **--profiling** 启用 web 界面查看 host:port/debug/pprof/ (default true)



## Authentication flags

认证配置

- **--anonymous-auth** 启用对API服务器的安全端口的匿名请求。 未被其他身份验证方法拒绝的请求被视为匿名。 匿名请求的用户名是system：anonymous，组名是system：unauthenticated。 （默认为true）
- **--api-audiences strings** API的标识符。 ServiceAccount token 验证者将验证针对API使用的令牌是否已绑定到这些受众中的至少一个。 如果配置了--service-account-issuer标志，但未配置此标志，则此字段默认为包含发布者URL的单个元素列表。
- **--authentication-token-webhook-cache-ttl duration** 缓存来自Webhook token 身份验证器的响应的持续时间。 （默认为2m0s）
- **--authentication-token-webhook-config-file string** 具有webhook配置的文件，用于以kubeconfig格式进行 token 认证。 API服务器将查询远程服务，以确定bearer token 的身份验证。
- **--authentication-token-webhook-version string**  the authentication.k8s.io TokenReview 使用的 API 版本
- **--client-ca-file string** 如果已设置，则使用与客户端证书的CommonName对应的标识对任何提出由client-ca文件中的授权机构之一签名的客户端证书的请求进行身份验证。
- **--enable-bootstrap-token-auth** 启用以允许将 kube-system 名称空间中类型为  bootstrap.kubernetes.io/token 的 Secret 用于TLS引导身份验证。
- **--oidc-ca-file string** 如果设置，则OpenID服务器的证书将由oidc-ca文件中的授权机构之一验证，否则将使用主机的根CA设置。
- **--oidc-client-id string** 如果设置了oidc-issuer-url，则必须设置OpenID Connect客户端的客户端ID。
- **--oidc-groups-claim string** 如果提供的话，用于指定用户组的定制OpenID Connect声明的名称。 声明值应为字符串或字符串数组。 此标志是实验性的，请参阅身份验证文档以获取更多详细信息。









## Authorization flags

授权配置 



## Cloud provider flags

- **--cloud-config** 云厂商的配置文件
- **--cloud-provider** 云厂商



## API enablement flags

**--runtime-config mapStringString** ，一个键值对Map，支持的选项如下：

v1=true|false for the core API group
<group>/<version>=true|false for a specific API group and version (e.g. apps/v1=true)
api/all=true|false controls all API versions
api/ga=true|false controls all API versions of the form v[0-9]+
api/beta=true|false controls all API versions of the form v[0-9]+beta[0-9]+
api/alpha=true|false controls all API versions of the form v[0-9]+alpha[0-9]+
api/legacy is deprecated, and will be removed in a future version



## Egress selector flags

**--egress-selector-config-file string**

带有apiserver出口选择器配置的文件。



## Admission flags



## Metrics flags

指标选项

**--show-hidden-metrics-for-version string**

您要为其显示隐藏指标的先前版本。 仅先前的次要版本有意义，将不允许其他值。 格式为<major>。<minor>，例如：“ 1.16”。 这种格式的目的是确保您有机会注意到下一个发行版是否隐藏了其他指标，而不是在以后将其永久删除时感到惊讶。



## Misc flags

杂项

- **--allow-privileged** 是否允许特权模式 ，calico 需要运行在特权模式
- **--apiserver-count int** 群集中运行的apiserver的数量，必须为正数。 （在启用--endpoint-reconciler-type=master-count时使用。）（默认值为1）
- **--enable-aggregator-routing** 打开到端点IP而不是集群IP的聚合器路由请求。
- **--endpoint-reconciler-type string** 使用哪种 reconciler， (master-count, lease, none) (default "lease")
- **--event-ttl duration ** 事件保留的时间，默认为 1 小时
- **--kubelet-certificate-authority string** kubelet 使用的 CA 文件
- **--kubelet-client-certificate string** kubelet 使用的证书文件
- **--kubelet-client-key string** kubelet 使用的私钥文件
- **--kubelet-https** 访问 kubelet 时使用 https，默认为 true
- **--kubelet-preferred-address-types strings** 用于kubelet连接的首选NodeAddressTypes列表，(default [Hostname,InternalDNS,InternalIP,ExternalDNS,ExternalIP])
- **--kubelet-timeout duration** kubelet 操作的超时时间
- **--kubernetes-service-node-port int** 如果非零，那么Kubernetes主服务（由apiserver创建/维护）将是NodePort类型，使用它作为端口的值。 如果为零，则Kubernetes主服务将为ClusterIP类型。
- **--max-connection-bytes-per-sec int ** 如果不为零，则将每个用户连接限制为该字节数/秒。 当前仅适用于长时间运行的请求。
- **--proxy-client-cert-file string** 通过代理访问 apiserver 时用到的证书文件
- **--proxy-client-key-file string** 通过代理访问 apiserver 时用到的私钥文件
- **--service-account-signing-key-file string** Path to the file that contains the current private key of the service account token issuer. The issuer will sign issued ID tokens with this private key. (Requires the 'TokenRequest' feature gate.)
- **--service-cluster-ip-range string ** service 用到的 ip 范围 
- **--service-node-port-range portRange** NodePort 使用到的端口范围



## Global flags

- **--add-dir-header ** If true, adds the file directory to the header
- **--alsologtostderr** log to standard error as well as files
- **-h, --help** 查看帮助 
- **--log-backtrace-at traceLocation** when logging hits line file:N, emit a stack trace (default :0)
- **--log-dir string** 日志目录
- **--log-file string** 日志文件
- **--log-file-max-size uint** 定义日志文件可以增长到的最大大小。 单位为兆字节。 如果值为0，则最大文件大小为无限制。 （默认为1800）
- **--log-flush-frequency duration** 日志刷新的时间间隔，默认 5 秒
- **--logtostderr** log to standard error instead of files (default true)
- **--skip-headers** If true, avoid header prefixes in the log messages
- **--skip-log-headers** If true, avoid headers when opening log files
- **--stderrthreshold severity** logs at or above this threshold go to stderr (default 2)
- **-v, --v Level** 日志等级
- **--version version[=true]** 查看版本
- ** --vmodule moduleSpec**  以逗号分隔的pattern = N设置列表，用于文件过滤的日志记录





