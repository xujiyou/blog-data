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

```
$ etcdctl put foo bar --cacert=/etc/etcd/cert/ca.pem
OK
$ etcdctl get foo --cacert=/etc/etcd/cert/ca.pem
foo
bar
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



## 部署 kube-apiserver







