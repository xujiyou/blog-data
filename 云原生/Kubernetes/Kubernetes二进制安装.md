# Kubernetes 二进制安装

首先，集群中每个节点 Docker 是安装好的，并且其储存也是挂在一个外部的盘中的，并且没有正在运行中的容器。

各个节点之间的 admin 账户都是配置了免密的。

SELinux 和防火墙都是关闭的。

每个节点下， proxychains4 代理都可以使用，

各项环境如下：

```bash
$ cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
172.20.20.162 fueltank-1.cloud.bbdops.com fueltank-1
172.20.20.179 fueltank-2.cloud.bbdops.com fueltank-2
172.20.20.145 fueltank-3.cloud.bbdops.com fueltank-3 
172.20.20.222 fueltank-4.cloud.bbdops.com fueltank-4
172.20.20.218 fueltank-5.cloud.bbdops.com fueltank-5
172.20.20.233 fueltank-6.cloud.bbdops.com fueltank-6

$ cat /etc/redhat-release 
CentOS Linux release 7.6.1810 (Core) 

$ uname -a
Linux fueltank-1.cloud.bbdops.com 3.10.0-957.27.2.el7.x86_64 #1 SMP Mon Jul 29 17:46:05 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux

$ docker --version
Docker version 19.03.5, build 633a0ea
```



---



### 第一步：安装 etcd 集群（静态发现）

kuberntes 集群使用 etcd 存储所有数据，是最重要的组件之一，注意 etcd集群需要奇数个节点(1,3,5...)，本文档使用3个节点做集群。

官网并没有安装 etcd 集群的方法。。。

只能东拼西凑一下了，回头详细学习下 etcd 官方文档。

---

以下每个节点都要做。

首先 yum 安装 etcd：

```bash
$ sudo yum install etcd -y
```

设置 etcd api 版本：

```bash
$ echo "export ETCDCTL_API=3" >> ~/.bash_profile
$ source ~/.bash_profile
```

检查安装是否完成：

```bash
$ etcd --version
etcd Version: 3.3.11
Git SHA: 2cf9e51
Go Version: go1.10.3
Go OS/Arch: linux/amd64
$ etcdctl version
etcdctl version: 3.3.11
API version: 3.3
```

yum 帮我们做了很多事情，比如说 etcd 的证书和私钥文件在 yum 安装时就生成好了：

```bash
$ ls /etc/etcd/cert/
ca.pem  etcd-key.pem  etcd.pem
```

同时，yum 也帮我们创建了一个名为 etcd 的用户：

```bash
$ cat /etc/passwd | grep etcd
etcd:x:998:995:etcd user:/var/lib/etcd:/sbin/nologin
```

下面创建 etcd 的储存目录，把数据放在外挂盘上，并配置新目录的权限，然后把 etcd 用户加入到 admin 用户组：

```bash
$ sudo mkdir /mnt/vde/etcd
$ sudo chown -R admin:admin /mnt/vde/etcd
$ sudo chmod 775 /mnt/vde/etcd
$ sudo usermod -aG admin etcd
```

同时，对俩密钥文件也需要更改权限，让 etcd 用户可以访问：

```bash
$ sudo chown -R admin:admin /etc/etcd/cert
$ sudo chmod 664 /etc/etcd/cert/etcd-key.pem
$ sudo chmod 664 /etc/etcd/cert/etcd.pem 
$ sudo chmod 664 /etc/etcd/cert/ca.pem
```



关于配置，官网并没有讲 yum 方式的配置，但是官网推荐通过 yum 方式安装。

发现这篇讲配置讲的不错：https://medium.com/@uzzal2k5/etcd-etcd-cluster-configuration-for-kubernetes-779455337db6

修改 etcd 配置文件 `/etc/etcd/etcd.conf` ,以 fueltank-1 为例，其他两台以同样的套路修改。

注意，这里只能是 IP 地址，不能是域名或 hostname

```properties
#[Member]
ETCD_DATA_DIR="/mnt/vde/etcd" # 数据储存地址
ETCD_LISTEN_PEER_URLS="http://172.20.20.162:2380" # 用于和其他节点通讯的地址
ETCD_LISTEN_CLIENT_URLS="https://172.20.20.162:2379,https://127.0.0.1:2379" # 监听客户端的地址
ETCD_NAME="fueltank-1" # 节点名称
#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://172.20.20.162:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://172.20.20.162:2379"
ETCD_INITIAL_CLUSTER="fueltank-1=http://172.20.20.162:2380,fueltank-2=http://172.20.20.179:2380,fueltank-3=http://172.20.20.145:2380"
#[Security]
ETCD_CERT_FILE="/etc/etcd/cert/etcd.pem"
ETCD_KEY_FILE="/etc/etcd/cert/etcd-key.pem"
```

可以看到，2379 端口都使用了 https，2380都使用的是 http。这是让客户端访问时用 https，集群之间访问时用 http。

然后在每台节点上启动 etcd，最好是都配置完成后，统一启动：

```bash
$ sudo systemctl enable etcd
$ sudo systemctl start etcd
```

如果出错了，可以用 `sudo journalctl -xe` 查看原因。

验证集群：

```bash
$ etcdctl put foo bar --cacert=/etc/etcd/cert/ca.pem
OK
$ etcdctl get foo --cacert=/etc/etcd/cert/ca.pem
foo
bar
```

查看 etcd 成员：

```bash
$ etcdctl member list --cacert=/etc/etcd/cert/ca.pem
87664c3cc645be22, started, fueltank-1, http://172.20.20.162:2380, https://172.20.20.162:2379
8cf2a5bef867d7cf, started, fueltank-2, http://172.20.20.179:2380, https://172.20.20.179:2379
a8070c86c64102fa, started, fueltank-3, http://172.20.20.145:2380, https://172.20.20.145:2379
```



完美！！！

错误处理。服务虽然起来了，但 journalctl -xe 一直报错：

```
etcd request cluster ID mismatch (got a want b)
```

这是因为之前各种纠错，`/mnt/vde/etcd` 目录有了错误数据导致的，先停掉服务，再删掉这个目录重建，然后再重启服务就好了。

这次安装 etcd 集群的姿势还是比较正确的。。。

etcd 集群搞定之后，下一步安装 k8s 的各个组件，最后安装 Calico 和 CoreDNS。

关于 k8s 各组件的作用可以参考  [核心组件运行原理.md](核心组件运行原理.md) 



---



## 第二步：安装 kube-apiserver

目前 yum 还不能很好的安装这些组件。只能自己动手，丰衣足食了。

先下载各项组件：https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.17.md#downloads-for-v1173

分别下载 server，node，client 三个压缩包。下载完成解压。

将 kube-apiserver 加入 PATH：

```bash
$ cp kube-apiserver /usr/bin/kube-apiserver
```

编辑 systemd 服务文件：

```
$ sudo vim /usr/lib/systemd/system/kube-apiserver.service
[Unit]
Description=Kubernetes API Server
After=etcd.service
Wants=etcd.service

[Service]
EnvironmentFile=/etc/kubernetes/apiserver
ExecStart=/usr/bin/kube-apiserver $KUBE_API_ARGS
Restart=on-failure
Type=notify
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

生成 kube-apiserver 的证书及私钥，参考：https://github.com/coreos/coreos-kubernetes/blob/master/Documentation/openssl.md

```bash
$ openssl genrsa -out ca-key.pem 2048
$ openssl req -x509 -new -nodes -key ca-key.pem -days 10000 -out ca.pem -subj "/CN=kube-ca"

# 生成 kube-apiserver 的证书及私钥
$ mkdir kube-apiserver & cd kube-apiserver
$ vim openssl.cnf
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = kubernetes
DNS.2 = kubernetes.default
DNS.3 = kubernetes.default.svc
DNS.4 = kubernetes.default.svc.cluster.local
IP.1 = 172.20.20.162
IP.2 = 172.20.20.179
IP.3 = 172.20.20.145

$ openssl genrsa -out apiserver-key.pem 2048
$ openssl req -new -key apiserver-key.pem -out apiserver.csr -subj "/CN=kube-apiserver" -config openssl.cnf
$ openssl x509 -req -in apiserver.csr -CA ../ca.pem -CAkey ../ca-key.pem -CAcreateserial -out apiserver.pem -days 365 -extensions v3_req -extfile openssl.cnf

# 生成 kubelet 的证书及私钥
$ cd ../
$ mkdir kubelet & cd kubelet
$ vim openssl.cnf
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
IP.1 = 127.0.0.1
IP.2 = 172.20.20.162
IP.3 = 172.20.20.179
IP.4 = 172.20.20.145

$ openssl genrsa -out kubelet-key.pem 2048
$ openssl req -new -key kubelet-key.pem -out kubelet.csr -subj "/CN=kubelet" -config openssl.cnf
$ openssl x509 -req -in kubelet.csr -CA ../ca.pem -CAkey ../ca-key.pem -CAcreateserial -out kubelet.pem -days 365 -extensions v3_req -extfile openssl.cnf
```

编辑配置文件：

```
$ sudo vim /etc/kubernetes/apiserver
KUBE_API_ARGS=" \
--admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota,NodeRestriction,MutatingAdmissionWebhook,ValidatingAdmissionWebhook \
--apiserver-count=1 \
--allow-privileged=true \
--audit-log-maxage=30 \
--audit-log-maxbackup=3 \
--audit-log-maxsize=100 \
--audit-log-path=/mnt/vde/kube-apiserver \
--authorization-mode=Node,RBAC \
--anonymous-auth=false \
--etcd-cafile=/etc/etcd/cert/ca.pem \
--etcd-certfile=/etc/etcd/cert/etcd.pem \
--etcd-keyfile=/etc/etcd/cert/etcd-key.pem \
--etcd-servers=https://127.0.0.1:2379 \
--kubelet-https=true \
--kubelet-certificate-authority=/home/admin/k8s-cluster/cert/ca.pem \
--kubelet-client-certificate=/home/admin/k8s-cluster/cert/kubelet/kubelet.pem \
--kubelet-client-key=/home/admin/k8s-cluster/cert/kubelet/kubelet-key.pem \
--enable-swagger-ui=true \
--client-ca-file=/home/admin/k8s-cluster/cert/ca.pem \
--tls-cert-file=/home/admin/k8s-cluster/cert/kube-apiserver/apiserver.pem \
--tls-private-key-file=/home/admin/k8s-cluster/cert/kube-apiserver/apiserver-key.pem \
--enable-aggregator-routing=true \
"
```

关于 kube-apiserver 的配置：https://kubernetes.io/zh/docs/reference/command-line-tools-reference/kube-apiserver/

我数了下，配置项也不多，也就 96 个的样子。。。

启动服务：

```
$ sudo systemctl enable kube-apiserver.service
$ sudo systemctl start kube-apiserver.service
```

检查 kube-apiserver 运行状态：

```
$ sudo systemctl status kube-apiserver
```



## 第三步：安装 kube-controller-manager

kube-controller-manager 也是安装在 master 节点的。

将 kube-controller-manager  放到 PATH 中：

```bash
$ sudo cp kube-controller-manager /usr/bin/
```

创建 service 文件：

```
$ sudo vim /usr/lib/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
After=kube-apiserver.service
Wants=kube-apiserver.service

[Service]
EnvironmentFile=/etc/kubernetes/kube-controller-manager
ExecStart=/usr/bin/kube-controller-manager $KUBE_CONTROLLER_MANAGER_ARGS
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

kube-controller-manager 的配置参考：https://kubernetes.io/zh/docs/reference/command-line-tools-reference/kube-controller-manager/

编写配置文件：

```
$ sudo vim /etc/kubernetes/kube-controller-manager
KUBE_CONTROLLER_MANAGER_ARGS=" \
--address=127.0.0.1 \
--master=http://127.0.0.1:8080 \
--allocate-node-cidrs=true \
--service-cluster-ip-range=10.43.0.0/16 \
--cluster-cidr=10.42.0.0/16 \
--cluster-name=kubernetes \
--cluster-signing-cert-file=/home/admin/k8s-cluster/cert/ca.pem \
--cluster-signing-key-file=/home/admin/k8s-cluster/cert/ca-key.pem \
--service-account-private-key-file=/home/admin/k8s-cluster/cert/ca-key.pem \
--root-ca-file=/home/admin/k8s-cluster/cert/ca.pem \
--leader-elect=true \
"
```

启动服务：

```
$ sudo systemctl enable kube-controller-manager.service
$ sudo systemctl start kube-controller-manager.service
```



## 第四步：安装 kube-scheduler

kube-scheduler 也是运行在 master 上的。

先把 kube-scheduler 放到 PATH：

```
$ sudo cp kube-scheduler /usr/bin/
```

创建 service 文件：

```
$ sudo vim /usr/lib/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes Scheduler
After=kube-apiserver.service
Wants=kube-apiserver.service

[Service]
EnvironmentFile=/etc/kubernetes/kube-scheduler
ExecStart=/usr/bin/kube-scheduler $KUBE_SCHEDULER_ARGS
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

创建配置文件，可参考官方配置：https://kubernetes.io/zh/docs/reference/command-line-tools-reference/kube-scheduler/

```
$ sudo vim /etc/kubernetes/kube-scheduler
KUBE_SCHEDULER_ARGS=" \
--address=127.0.0.1 \
--master=http://127.0.0.1:8080 \
--leader-elect=true \
"
```

启动服务：

```
$ sudo systemctl enable kube-scheduler.service
$ sudo systemctl start kube-scheduler.service
```



## 第五步：安装 kubelet

kubelet 是运行在 node 节点的，不过这里为了节约机器，先把它安装在 master 节点。

先把 kubelet 放到 PATH 之中：

```bash
$ sudo cp kubelet /usr/bin/
```

创建 service：

```
$ sudo vim /usr/lib/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=docker.service
Requires=docker.service
 
[Service]
EnvironmentFile=/etc/kubernetes/kubelet
ExecStart=/usr/bin/kubelet $KUBELET_ARGS
Restart=on-failure
RestartSec=5
KillMode=process
 
[Install]
WantedBy=multi-user.target
```

创建配置文件，kubelet 的配置可以参考：https://kubernetes.io/zh/docs/reference/command-line-tools-reference/kubelet/

```
$ sudo vim /etc/kubernetes/kubelet

```

