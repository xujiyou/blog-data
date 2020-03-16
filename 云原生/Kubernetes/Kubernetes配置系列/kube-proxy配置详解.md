# kube-proxy 配置详解

官方文档地址：https://kubernetes.io/zh/docs/reference/command-line-tools-reference/kube-proxy/

共 51 个配置选项。

#### --add-dir-header

If true, adds the file directory to the header



#### --alsologtostderr 

日志重定向到标准错误和文件



#### --bind-address 0.0.0.0

Kube-proxy 监听的地址，默认就是 0.0.0.0



#### --cleanup

如果为 true，就会清理 iptables 和 ipvs 然后退出



#### --cluster-cidr string

Pod 的IP地址范围



#### --config string

配置文件的路径



#### --config-sync-period duration

多久从 kube-apiserver 刷新一次配置，必须大于0，默认为 15分钟。



#### --conntrack-max-per-core int32 

每个 CPU 的最大 NAT 连接数量，0 代表无限，默认是 32768



#### --conntrack-min int32

最小的 NAT 连接数量，默认 131072 ，--conntrack-max-per-core 为 0 时，这个参数将无效



#### --conntrack-tcp-timeout-close-wait duration

TCP 连接的 NAT 超时时间，即 TCP 处于 CLOSE_WAIT 状态的时间，默认为 1小时



#### --conntrack-tcp-timeout-established duration

在 established 状态的超时时间，默认 24 小时。



#### --feature-gates mapStringBool

特性开关



#### --healthz-bind-address 0.0.0.0

健康检查绑定的 IP 地址



#### --healthz-port int32 

健康检查的端口，默认 10256，为 0 则禁用。



#### --hostname-override string

如果不为空，则覆盖时间的主机名



#### --iptables-masquerade-bit int32

取值必须在 [0, 31]之间 (default 14)。

如果使用纯 iptables 代理，标记需要SNAT的包的fwmark空间的位。



#### --iptables-min-sync-period duration

最小的时间间隔去刷新 endpoints 和 service 的规则，比如'5s', '1m', '2h22m'



#### --iptables-sync-period duration

iptables 规则刷新的最大时间间隔，默认 30s。



#### --ipvs-exclude-cidrs strings

一个逗号分隔的CIDR列表，ipvs代理在清理ipvs规则时不应接触该列表。



#### --ipvs-min-sync-period duration

Itvs最小的时间间隔去刷新 endpoints 和 service 的规则，比如'5s', '1m', '2h22m'



#### --ipvs-scheduler string

当代理模式是 ipvs 时使用 ipvs 调度



#### --ipvs-strict-arp

开启静态 ARP，arp_ignore 为 1 ， arp_announce 为 2



#### --ipvs-sync-period duration

Ipvs 规则刷新的最大时间间隔，默认 30s。



#### --kube-api-burst int32

默认 10，与 kube-apiserver 交流。



#### --kube-api-content-type string

请求 apiserver 的数据类型，默认 application/vnd.kubernetes.protobuf



#### --kube-api-qps float32

与kubernetes apiserver交谈时使用的QPS，默认 5。

QPS：每秒查询率QPS是对一个特定的查询服务器在规定时间内所处理流量多少的衡量标准，在因特网上，作为域名系统服务器的机器的性能经常用每秒查询率来衡量。



#### --kubeconfig string

kubeconfig 配置地址，要给予kube-proxy 一定的 apiserver 的权限。



#### --log-backtrace-at traceLocation

默认 0，当处于哪行时，开始记录日志



#### --log-dir string

日志目录



#### --log-file string

日志文件



#### --log-file-max-size uint

日志文件最大大小



#### --log-flush-frequency duration

日志过多少时间就刷到文件中，默认 5 秒。



#### --logtostderr

错误日志传到标准错误，而不是文件，默认为true



#### --masquerade-all

If using the pure iptables proxy, SNAT all traffic sent via Service cluster IPs (this not commonly needed)



#### --master string

kube-apiserver 的地址，这个值会覆盖 kubeconfig 中的地址。



#### --metrics-bind-address 0.0.0.0

metrics 绑定的地址



#### --metrics-port int32

metrics 绑定的端口，默认 10249



#### --nodeport-addresses strings

NodePorts 可以使用的地址，为 空列表则表示所有集群中的节点地址。



#### --oom-score-adj int32

[-1000, 1000] (default -999)，进程点数，在发生内存溢出时，linux 会选择杀死哪些进程，点数越大，*这个进程越有可能被杀死*



#### --profiling

If true enables profiling via web interface on /debug/pprof handler



#### --proxy-mode ProxyMode

'userspace' (older) or 'iptables' (faster) or 'ipvs'



#### --proxy-port-range port-range

主机上的端口范围



#### --skip-headers

If true, avoid header prefixes in the log messages



#### --skip-log-headers

If true, avoid headers when opening log files



#### --stderrthreshold severity

logs at or above this threshold go to stderr (default 2)



#### --udp-timeout duration

How long an idle UDP connection will be kept open (e.g. '250ms', '2s').  Must be greater than 0. Only applicable for proxy-mode=userspace (default 250ms)



#### --version version[=true]

打印版本退出



#### --vmodule moduleSpec

comma-separated list of pattern=N settings for file-filtered logging



#### --write-config-to string

If set, write the default configuration values to this file and exit.





























