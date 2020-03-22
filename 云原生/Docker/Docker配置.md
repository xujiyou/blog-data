# Docker 配置

官方的配置文档：https://docs.docker.com/engine/reference/commandline/dockerd/

如果 docker 是使用 yum 装的，其 service 文件在 `/usr/lib/systemd/system/docker.service`，另外还有一个 `/usr/lib/systemd/system/docker.socket` 文件

从这个service 中了解到，其实是使用 `dockerd` 这个命令来启动 docker 守护程序的。

还需注意的是，在启动 dockerd 的时候，还自动启动了 `containerd.service`。这就说明了，docker 守护进程其实是通过 containerd 来完成任务的！！！



## 配置

#### --add-runtime runtime

注册其他OCI兼容的运行时（默认[]）



#### --allow-nondistributable-artifacts list

将不可分发的工件推送到指定的注册表（默认[]）



#### --api-cors-header string

设置 API 允许跨域 header



#### --authorization-plugin list 

要加载的授权插件（默认[]）



#### --bip string 

指定网桥IP



#### -b, --bridge string   

将容器连接到网桥



#### --cgroup-parent string   

为所有容器设置父级cgroup



#### --cluster-advertise string 

要对外使用的地址或接口名称



#### --cluster-store string 

分布式存储后端的URL



#### --cluster-store-opt map

设置集群存储选项（默认映射[]）



#### --config-file string

守护程序配置文件 (default "/etc/docker/daemon.json")



#### --containerd string

和 containerd 通信使用到的 socket 文件



#### --cpu-rt-period int

将CPU实时时间限制为多少微秒



#### --cpu-rt-runtime int

将CPU实时运行时间限制为多少微秒



#### --data-root string

持久化 Docker 状态的根目录，(default "/var/lib/docker")



#### -D, --debug

启用调试模式



#### --default-gateway ip  

容器默认网关IPv4地址



#### --default-gateway-v6 ip 

容器默认网关IPv6地址



#### --default-address-pool

设置本地节点网络的默认地址池



#### --default-runtime string 

容器的默认OCI运行时，(default "runc")



#### --default-ulimit ulimit

容器的默认ulimit（默认[]）



#### --dns list

要使用的DNS服务器（默认[]）



#### --dns-opt list 

使用的DNS选项（默认[]



#### --dns-search list

要使用的DNS搜索域（默认[]）



#### --exec-opt list 

运行时执行选项（默认为[]）



#### --exec-root string 

执行状态文件的根目录，(default "/var/run/docker")



#### --experimental 

启用实验功能



#### --fixed-cidr string

固定IP的IPv4子网



#### --fixed-cidr-v6 string 

固定IP的IPv6子网



#### -G, --group string

UNIX套接字组，(default "docker")



#### --help

查看帮助



#### -H, --host list

要连接的守护程序套接字（默认[]）



#### --icc

启用容器间通信（默认为true）



#### --init 

在容器中运行init以转发信号并获得进程



#### --init-path string 

docker-init二进制文件的路径



#### --insecure-registry list 

启用不安全的注册表通信（默认[]）



#### --ip ip 

绑定容器端口时的默认IP（默认0.0.0.0）



#### --ip-forward

启用net.ipv4.ip_forward（默认为true）



#### --ip-masq

启用IP伪装（默认为true）



#### --iptables 

启用iptables规则添加（默认为true）



#### --ipv6 

启用IPv6网络



#### --label list 

将键=值标签设置为守护程序（默认为[]）



#### --live-restore 

在容器仍在运行时启用Docker的实时还原



#### --log-driver string

容器日志的默认驱动程序（默认“ json-file”）



####  -l, --log-level string

Set the logging level ("debug", "info", "warn", "error", "fatal") (default "info")



#### --log-opt map

容器的默认日志驱动程序选项（默认映射[]）



#### --max-concurrent-downloads int 

设置每个请求的最大并发下载量（默认为3）



####  --max-concurrent-uploads int 

设置每次推送的最大同时上传数量（默认5）



#### --metrics-addr string

设置默认地址和端口以在其上提供指标API



#### --mtu int 

设置容器网络MTU



#### --node-generic-resources list 

公布用户定义的资源



#### --no-new-privileges 

默认情况下为新容器设置no-new-privileges



#### --oom-score-adjust int  

设置守护程序的oom_score_adj（默认值为-500）



#### -p, --pidfile string 

守护程序PID文件使用的路径，(default "/var/run/docker.pid")



#### --raw-logs

完整时间戳，无ANSI着色



#### --registry-mirror list

首选Docker注册表镜像（默认[]）



#### --seccomp-profile string

seccomp配置文件的路径



#### --selinux-enabled

启用selinux支持



#### --shutdown-timeout int

设置默认关机超时（默认15）



#### -s, --storage-driver string

使用的存储驱动程序



#### --storage-opt list

存储驱动程序选项（默认为[]）



#### --swarm-default-advertise-addr string

设置群集地址的默认地址或接口



####  --tls

使用TLS; --tlsverify



#### --tlscacert string

信任的 CA，默认 (default "~/.docker/ca.pem")



####  --tlscert string 

证书文件， (default "~/.docker/cert.pem")



#### --tlskey string

私钥文件，(default ~/.docker/key.pem")



#### --tlsverify 

使用TLS并验证远程连接



#### --userland-proxy 

使用Userland代理进行环回流量（默认为true）



#### --userland-proxy-path string

Userland代理二进制文件的路径



#### --userns-remap string

用户名称空间的用户/组设置



#### -v, --version      

打印版本然后退出



