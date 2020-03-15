# etcd 配置详解

官方文档：https://etcd.io/docs/v3.4.0/op-guide/configuration/

可以通过配置文件，各种命令行标志和环境变量来配置etcd。

配置文件是一种YAML文件，其名称和值由一个或多个下面描述的命令行标志组成。为了使用此文件，请将文件路径指定为 `--config-file` 或  `ETCD_CONFIG_FILE` 环境变量的值。 [示例配置文件](https://etcd.io/docs/v3.4.0/etcd.conf.yml.sample)可以被用作起始点根据需要创建一个新的配置文件。

在命令行上设置的选项优先于环境中的选项。如果提供了配置文件，则其他命令行标志和环境变量将被忽略。例如，`etcd --config-file etcd.conf.yml.sample --data-dir /tmp`将忽略该`--data-dir`标志。

标志的环境变量的格式 `--my-flag` 为 `ETCD_MY_FLAG` 。它适用于所有标志。

该 [官方使用的ETCD端口](http://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.txt)是 2379 和 2380，2379用于接收客户端请求，2380 用于 etcd 节点之间的对等通信。可以将etcd端口设置为接受TLS流量，非TLS流量，或同时接受TLS和非TLS流量。

要在Linux启动时使用自定义设置自动启动etcd，强烈建议使用 [systemd](http://freedesktop.org/wiki/Software/systemd/)单元。



## 配置

接下来介绍 etcd 所有的配置，共 65 项配置。etcd 的配置分成了几类，下面分类学习



### Member flags



#### –name

- 人类可读的 etcd 成员的名称。默认是 “default”
- default: “default”
- 环境变量名：ETCD_NAME
- 每个节点都应该拥有一个唯一的名字，Hostname 或者 machine-id 都是不错的选择，这个名字也应该用在 `--initial-cluster` 配置上，表示当前节点的名字，它也会用在动态服务发现上。



#### –data-dir

- 数据储存目录
- default: “${name}.etcd”
- 环境变量：ETCD_DATA_DIR



#### –wal-dir

- 专用wal目录的路径，如果设置了，etcd 会将 wal 数据存到这个目录中，而不是存到 --data-dir 指定的路径中。这允许使用专用磁盘，并有助于避免日志记录和其他IO操作之间的io竞争。
- default: ""
- 环境变量：ETCD_WAL_DIR



#### –snapshot-count

- 已提交事务数超过这个值就触发在磁盘上生成快照的程序。
- default: “100000”
- 环境变量：ETCD_SNAPSHOT_COUNT



#### –heartbeat-interval

- 心跳间隔时间（以毫秒为单位）。
- default: “100”
- 环境变量：ETCD_HEARTBEAT_INTERVAL



#### –election-timeout

- 选举超时的时间（以毫秒为单位）。
- default: “1000”
- 环境变量：ETCD_ELECTION_TIMEOUT



#### –listen-peer-urls

- 侦听 etcd 节点之间对等流量的URL列表，这个配置指定了要监听的地址和端口，用来在节点之间传输数据，默认是2380端口，可以使用 HTTP 或 HTTPS，也可以监听多个 IP。
- default: “http://localhost:2380”
- 环境变量：ETCD_LISTEN_PEER_URLS
- 注意：域名绑定无效，必须用 IP 地址



#### –listen-client-urls

- 作用和 `–listen-peer-urls` 差不多，只不过是用来监听客户端请求的。
- default: “http://localhost:2379”
- 环境变量：ETCD_LISTEN_CLIENT_URLS
- 注意：域名绑定无效，必须用 IP 地址



#### –max-snapshots

- 保留的快照文件最大数量（0为无限）
- default: 5
- 环境变量：ETCD_MAX_SNAPSHOTS
- Windows用户的默认设置是无限制的，建议手动清除到5（出于安全性的考虑）。



#### –max-wals

- 保留的wal文件最大数量（0是无限的）
- default: 5
- 环境变量：ETCD_MAX_WALS
- Windows用户的默认设置是无限制的，建议手动清除到5（出于安全性的考虑）。



#### –cors

- 以逗号分隔的CORS来源白名单（允许跨越的地址列表）。
- default: ""
- 环境变量：ETCD_CORS



#### –quota-backend-bytes

- 后端储存大小超过给定配额时引发警报（0默认为低空间配额）。
- default: 0
- 环境变量：ETCD_QUOTA_BACKEND_BYTES



#### –backend-batch-limit

- 提交后端事务之前的最大操作。（有点懵逼）
- default: 0
- 环境变量：ETCD_BACKEND_BATCH_LIMIT



#### –backend-bbolt-freelist-type

- etcd后端（bboltdb）使用的列表类型（支持数组和 Map）。
- default: 0
- 环境变量：ETCD_BACKEND_BATCH_INTERVAL



#### –max-txn-ops

- 最大的事务操作数量
- default: 128
- 环境变量：ETCD_MAX_TXN_OPS



#### –max-request-bytes

- 服务器将接受的最大客户端请求大小（以字节为单位）。
- default: 1572864
- 环境变量：ETCD_MAX_REQUEST_BYTES



#### –grpc-keepalive-min-time

- 客户端在ping服务器之前应等待的最小持续时间间隔。
- default: 5s
- 环境变量：ETCD_GRPC_KEEPALIVE_MIN_TIME



#### –grpc-keepalive-interval

- 服务器到客户端ping的频率持续时间，以检查连接是否有效（0禁用）。
- default: 2h
- 环境变量：ETCD_GRPC_KEEPALIVE_INTERVAL



#### –grpc-keepalive-timeout

- 连接超时时间（0为禁用）。
- default: 20s
- 环境变量：ETCD_GRPC_KEEPALIVE_TIMEOUT





### Clustering flags

`--initial-advertise-peer-urls`, `--initial-cluster`, `--initial-cluster-state`, 和 `--initial-cluster-token` 会在启动新成员时使用，在重启一个成员时不会使用。

`--discovery` 为前缀的配置用于动态服务发现，详见： [etcd动态发现.md](etcd动态发现.md) 



#### –initial-advertise-peer-urls

- 此成员的对等URL列表，用于发布到集群的其余部分。这些地址用于在集群之间传递etcd数据。所有集群成员必须至少有一个路由。这些URL可以包含域名。
- default: “http://localhost:2380”
- 环境变量：ETCD_INITIAL_ADVERTISE_PEER_URLS
- 例子: “[http://example.com](http://example.com/):2380, http://10.0.0.1:2380”



#### –initial-cluster

- 用于静态引导的初始群集配置。
- default: “default=http://localhost:2380”
- 环境变量：ETCD_INITIAL_CLUSTER
- 注意，key 应该是 --name 中指定的值 



#### –initial-cluster-state

- 初始集群状态(“new” 或 “existing”)，初始静态或DNS引导过程中存在的所有成员都设置为new，如果此选项设置为 existing ，则etcd将尝试加入现有集群，如果设置了错误的值，etcd将尝试启动，但会安全失败。
- default: “new”
- 环境变量：ETCD_INITIAL_CLUSTER_STATE



#### –initial-cluster-token

- 引导期间etcd集群的初始集群 token。
- default: “etcd-cluster”
- 环境变量：ETCD_INITIAL_CLUSTER_TOKEN



#### –advertise-client-urls

- 此成员的客户端可访问的URL列表。这些URL可以包含域名。
- default: “http://localhost:2379”
- 环境变量：ETCD_ADVERTISE_CLIENT_URLS
- example: “[http://example.com](http://example.com/):2379, http://10.0.0.1:2379”
- 请注意，如果从群集成员中发布诸如 http://localhost:2379 之类的URL，并且正在使用etcd的代理功能。这将导致循环，因为代理将向其自身转发请求，直到其资源（内存，文件描述符）最终耗尽为止。



#### –discovery

- 动态服务发现的 URL
- default: "”
- 环境变量：ETCD_DISCOVERY



#### –discovery-srv

- 用于引导群集的DNS srv域。
- default: "”
- 环境变量：ETCD_DISCOVERY_SRV



#### –discovery-srv-name

- 使用DNS引导时查询的DNS srv名称的后缀。
- default: "”
- 环境变量：ETCD_DISCOVERY_SRV_NAME



#### –discovery-fallback

- 当发现服务失败时时的预期行为 (“exit” or “proxy”) ，“proxy” 仅支持 v2 版本的 API
- default: “proxy”
- 环境变量：ETCD_DISCOVERY_FALLBACK



#### –discovery-proxy

- HTTP代理，用于发现服务的流量。
- default: "”
- 环境变量：ETCD_DISCOVERY_PROXY



#### –strict-reconfig-check

- 拒绝可能导致仲裁丢失的重新配置请求。
- default: 0
- 环境变量：ETCD_AUTO_COMPACTION_RETENTION



#### –auto-compaction-mode

- 值为 ‘periodic’ 或 ‘revision’，‘periodic’ 用于基于持续时间的保留，如果未提供时间单位，则默认为小时，‘revision’ 用于基于修订号的保留。
- default: periodic
- 环境变量：ETCD_AUTO_COMPACTION_MODE



#### –enable-v2

- 接收 V2 版本的客户端 的请求
- default: false
- 环境变量：ETCD_ENABLE_V2



## Proxy flags



“proxy” 仅支持 v2 版本的 API. 这里不学了。



## Security flags



#### –ca-file

- 过期



#### –cert-file

- 客户端使用的证书文件
- default: "”
- 环境变量：ETCD_CERT_FILE



#### –key-file

- 客户端使用的私钥文件
- default: "”
- 环境变量：ETCD_KEY_FILE



#### –client-cert-auth

- 开启客户端 TSL 认证
- default: false
- 环境变量：ETCD_CLIENT_CRL_FILE
- CA 认证不支持 gRPC 网关



#### –client-cert-allowed-hostname

- TLS 允许访问的地址
- default: "”
- 环境变量：ETCD_CLIENT_CERT_ALLOWED_HOSTNAME



#### –trusted-ca-file

- 信任的 CA
- default: "”
- 环境变量：ETCD_TRUSTED_CA_FILE



#### –auto-tls

- 客户端使用生成的证书
- default: false 
- 环境变量：ETCD_AUTO_TLS



#### –peer-ca-file

- 过期



#### –peer-cert-file

- 节点之间使用的证书
- default: "”
- 环境变量：ETCD_PEER_CERT_FILE



#### –peer-key-file

- 节点之间使用的私钥
- default: "”
- 环境变量：ETCD_PEER_KEY_FILE



#### –peer-client-cert-auth

- 节点之间是否开启证书认证
- default: false
- 环境变量：ETCD_PEER_CLIENT_CERT_AUTH



#### –peer-crl-file

- 对等证书吊销列表文件的路径。
- default: "”
- 环境变量：ETCD_PEER_CRL_FILE



#### –peer-trusted-ca-file

- 节点之间信任的 CA
- default: "”
- 环境变量：ETCD_PEER_TRUSTED_CA_FILE



#### –peer-auto-tls

- 节点对端使用生成的证书
- default: false
- 环境变量：ETCD_PEER_AUTO_TLS



#### –peer-cert-allowed-cn

- 允许使用CommonName进行对等身份验证
- default: "”
- 环境变量：ETCD_PEER_CERT_ALLOWED_CN



#### –peer-cert-allowed-hostname

- 节点 TLS 允许访问的地址
- default: "”
- 环境变量：ETCD_PEER_CERT_ALLOWED_HOSTNAME



#### –cipher-suites

- 服务器/客户端与对等方之间受支持的TLS密码套件的逗号分隔列表。
- default: "”
- 环境变量：ETCD_CIPHER_SUITES



## Logging flags



#### –logger

**Available from v3.4.** **WARNING: `--logger=capnslog` to be deprecated in v3.5.**

- 指定“ zap”用于结构化日志记录
- default: capnslog
- 环境变量：ETCD_LOGGER



#### –log-outputs

- 如果使用 systemd 的话，指定 ‘stdout’ or ‘stderr’  将会跳过 journald 的日志收集。它逗号分隔的输出目标列表。
- default: default
- 环境变量：ETCD_LOG_OUTPUTS



#### –log-level

**Available from v3.4.**

- 配置日志级别，支持 debug, info, warn, error, panic, or fatal.
- default: info
- 环境变量：ETCD_LOG_LEVEL



#### –debug

**WARNING: to be deprecated in v3.5.**

#### –log-package-levels

**WARNING: to be deprecated in v3.5.**





## Unsafe flags

使用不安全标志时请小心，因为它会破坏共识协议提供的保证。例如，如果群集中的其他成员仍然存在，可能会感到恐慌。使用这些标志时，请遵循说明。

#### –force-new-cluster

- 强制创建一个新的单成员群集。它提交配置更改，以强制删除集群中的所有现有成员并添加自身，但是强烈建议不要这样做。
- default: false
- 环境变量：ETCD_FORCE_NEW_CLUSTER





## Miscellaneous flags

杂项

#### –version

- 打印版本并退出
- default: false



#### –config-file

- 从文件加载服务器配置。请注意，如果提供了配置文件，则其他命令行标志和环境变量将被忽略。
- default: "”
- 环境变量：ETCD_CONFIG_FILE



## Profiling flags

#### –enable-pprof

- 通过HTTP服务器启用运行时分析数据。地址位于客户端URL +“ / debug / pprof /”
- default: false
- 环境变量：ETCD_ENABLE_PPROF



#### –metrics

- 设置导出指标的详细程度，指定“extensive”以包括服务器端grpc直方图指标。
- default: basic
- 环境变量：ETCD_METRICS



#### –listen-metrics-urls

- 要侦听的其他URL列表，这些URL将同时响应/ metrics和/ health端点
- default: "”
- 环境变量：ETCD_LISTEN_METRICS_URLS





## Auth flags

#### –auth-token

- 指定一个用于访问 v3 版本的 token 策略，('simple' or 'jwt')
- default: “simple”
- 环境变量：ETCD_AUTH_TOKEN



#### –bcrypt-cost

- 指定用于哈希认证密码的bcrypt算法的成本/强度。有效值在4到31之间。
- default: 10



## Experimental flags

实验性的，不了解了。



#### 

















