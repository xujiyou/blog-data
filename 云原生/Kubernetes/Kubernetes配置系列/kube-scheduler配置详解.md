# kube-scheduler 配置详解

官方文档：https://kubernetes.io/zh/docs/reference/command-line-tools-reference/kube-scheduler/

共 66 个配置项，15 个过期配置项

## 杂项

#### --config string

配置文件地址



#### --master string

kube-apiserver 的地址，会覆盖 kubeconfig 中的地址。



#### --write-config-to string

如果设置了，会将配置写入这个文件并退出。



## 安全配置

#### --bind-address ip

监听的地址，默认 0.0.0.0



#### --cert-dir string

证书及私钥目录



#### --http2-max-streams-per-connection int

HTTP 流的最大连接数量，为 0 以为着使用 golang 默认的。



#### --secure-port int 

kube-scheduler 绑定的安全端口，默认10259



#### --tls-cert-file string

证书文件



#### --tls-cipher-suites strings

密码套件列表，如果省略，则使用 golang的默认列表，可能的值为：

TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA,TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256,TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA,TLS_ECDHE_ECDS
A_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_ECDSA_WITH_RC4_128_SHA,TLS_ECDHE_RSA_WITH_3DES_EDE_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA,TLS_ECDHE_RSA_W
ITH_AES_128_CBC_SHA256,TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305,TLS_ECDHE_RSA
_WITH_RC4_128_SHA,TLS_RSA_WITH_3DES_EDE_CBC_SHA,TLS_RSA_WITH_AES_128_CBC_SHA,TLS_RSA_WITH_AES_128_CBC_SHA256,TLS_RSA_WITH_AES_128_GCM_SHA256,TLS_RSA_WITH_AES_256_CBC_SHA,TLS_RSA_WITH_A
ES_256_GCM_SHA384,TLS_RSA_WITH_RC4_128_SHA



#### --tls-min-version string

TLS 最小版本，可选值为 VersionTLS10, VersionTLS11, VersionTLS12, VersionTLS13



####  --tls-private-key-file string

私钥文件



#### --tls-sni-cert-key namedCertKey 

A pair of x509 certificate and private key file paths, optionally suffixed with a list of domain patterns which are fully qualified domain names, possibly with prefixed wildcard segments. If no domain patterns are provided, the names of the certificate are extracted. Non-wildcard matches trump over wildcard matches, explicit domain patterns trump over extracted names. For multiple key/certificate pairs, use the --tls-sni-cert-key multiple times. Examples: "example.crt,example.key" or "foo.crt,foo.key:*.foo.com,foo.com". (default [])



## 认证配置

#### --authentication-kubeconfig string

认证用的 kubeconfig 文件



#### --authentication-skip-lookup

如果为 false，服务就会自己查找 kubeconfig 文件



#### --authentication-token-webhook-cache-ttl duration

The duration to cache responses from the webhook token authenticator. (default 10s)



#### --authentication-tolerate-lookup-failure

如果为true，则从群集查找缺少的身份验证配置的失败不会被认为是致命的。请注意，这可能导致将所有请求视为匿名的身份验证



#### --client-ca-file string

如果已设置，则任何呈现由客户端ca文件中的一个颁发机构签名的客户端证书的请求都将使用与客户端证书的公用名相对应的标识进行身份验证。



#### --requestheader-allowed-names strings

服务允许的 ip 和域名，如果这里不设置，也会使用证书里面设置的。



#### --requestheader-client-ca-file string

根证书CA 文件呢，用于在信任由--requestheader username headers指定的头中的用户名之前验证传入请求的客户端证书



#### --requestheader-extra-headers-prefix strings

List of request header prefixes to inspect. X-Remote-Extra- is suggested. (default [x-remote-extra-])

要检查的请求头前缀列表。建议使用X-Remote-Extra。（默认值[x-remote-extra-]）



#### --requestheader-group-headers strings

List of request headers to inspect for groups. X-Remote-Group is suggested. (default [x-remote-group])



#### --requestheader-username-headers strings 

List of request headers to inspect for usernames. X-Remote-User is common. (default [x-remote-user])



## 授权配置

#### --authorization-always-allow-paths strings

 (default [/healthz])，会跳过授权的一组地址，也会被 kube-apiserveer 跳过授权



#### --authorization-kubeconfig string

用于认证的 kube-config



#### --authorization-webhook-cache-authorized-ttl duration

The duration to cache 'authorized' responses from the webhook authorizer. (default 10s)



#### --authorization-webhook-cache-unauthorized-ttl duration

The duration to cache 'unauthorized' responses from the webhook authorizer. (default 10s)





## Leader 选举配置

#### --leader-elect

默认为 true，在启动之前会先进行 Leader 选举，这会是组件组成高可用集群。



#### --leader-elect-lease-duration duration

多少时间后，再进行选举。默认 15 秒。



#### --leader-elect-renew-deadline duration

Leader 在停止前尝试更新其余对等组件身份的间隔。这必须小于或等于 --leader-elect-lease-duration 定义的期限。这仅在启用 Leader 选举时适用。（默认10s）



#### --leader-elect-resource-lock endpoints

在 Leader 选举期间用于锁定的资源对象的类型，默认为 “endpointsleases”，



#### --leader-elect-resource-name string

在 Leader 选举期间用于锁定的资源对象的名称。默认为 “kube-scheduler”



#### --leader-elect-resource-namespace string

在 Leader 选举期间用于锁定的资源对象的命名空间。默认为 “kube-system”



#### --leader-elect-retry-period duration

客户应在尝试获取和更新领导之间等待的时间。 仅在启用了领导者选举的情况下才适用。默认 2 s。



## 全局选项

#### --add-dir-header

 If true, adds the file directory to the header



#### --alsologtostderr

log to standard error as well as files



####  -h, --help

查看帮助



#### --log-backtrace-at traceLocation 

 when logging hits line file:N, emit a stack trace (default :0)



#### -log-dir string

日志目录



#### --log-file string

日志文件



#### --log-file-max-size uint

日志最大大小，默认 1800 MB



#### --log-flush-frequency duration

日志刷新时间，默认 5 秒。



#### --logtostderr 

log to standard error instead of files (default true)



#### --skip-headers

 If true, avoid header prefixes in the log messages



#### --skip-log-headers

 If true, avoid headers when opening log files



#### --stderrthreshold severity 

logs at or above this threshold go to stderr (default 2)



####   -v, --v Level

日志等级



#### --version version[=true]

打印版本



#### --vmodule moduleSpec

comma-separated list of pattern=N settings for file-filtered logging