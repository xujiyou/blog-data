# 使用CRI-O 和 Kubeadm 搭建高可用 Kubernetes 集群

准备在 CentOS 8 中部署 CRI-O，然后用 Kuberadm 部署高可用 Kubernetes 集群。



## CRI-O 安装

在 Github 中下载 CRI-O 的二进制压缩包：https://storage.googleapis.com/k8s-conform-cri-o/artifacts/crio-v1.19.0.tar.gz

下载完成后进行解压。

在下载的二进制压缩包中有一个 README.md，可以按照其中的步骤进行安装。

安装：

```bash
$ sudo yum install make -y
$ sudo make install
```

过程如下：

```
install  -d -m 755 /etc/cni/net.d
install  -D -m 755 -t /opt/cni/bin cni-plugins/*
install  -D -m 644 -t /etc/cni/net.d contrib/10-crio-bridge.conf
install  -D -m 755 -t /usr/local/bin bin/conmon
install  -d -m 755 /usr/local/share/bash-completion/completions
install  -d -m 755 /usr/local/share/fish/completions
install  -d -m 755 /usr/local/share/zsh/site-functions
install  -d -m 755 /etc/containers
install  -D -m 755 -t /usr/local/bin bin/crio-status
install  -D -m 755 -t /usr/local/bin bin/crio
install  -D -m 644 -t /etc etc/crictl.yaml
install  -D -m 644 -t /usr/local/share/oci-umount/oci-umount.d etc/crio-umount.conf
install  -D -m 644 -t /etc/crio etc/crio.conf
install  -D -m 644 -t /usr/local/share/man/man5 man/crio.conf.5
install  -D -m 644 -t /usr/local/share/man/man5 man/crio.conf.d.5
install  -D -m 644 -t /usr/local/share/man/man8 man/crio.8
install  -D -m 644 -t /usr/local/share/bash-completion/completions completions/bash/crio
install  -D -m 644 -t /usr/local/share/fish/completions completions/fish/crio.fish
install  -D -m 644 -t /usr/local/share/zsh/site-functions completions/zsh/_crio
install  -D -m 644 -t /etc/containers contrib/policy.json
install  -D -m 644 -t /usr/local/lib/systemd/system contrib/crio.service
install  -D -m 755 -t /usr/local/bin bin/crictl
install  -D -m 755 -t /usr/local/bin bin/pinns
install  -D -m 755 -t /usr/local/bin bin/runc
install  -D -m 755 -t /usr/local/bin bin/crun
```

上边都是一些关键的文件和配置目录。

启动：

```bash
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now crio
$ sudo systemctl start --now crio
```

查看状态：

```bash
$ sudo systemctl status crio
```

如果要卸载，需要在解压的 CRI-O 目录中执行：

```bash
$ sudo make uninstall
```

测试：

```bash
$ sudo su -
$ export PATH=$PATH:/usr/local/bin
$ crictl ps
```

使用 `unix:///var/run/crio/crio.sock` 进行通信。

修改 `/etc/crio/crio.conf` 中的配置：

```
insecure_registries = ["registry.prod.bbdops.com"]
pause_image = "registry.prod.bbdops.com/common/pause:3.2"
```

因为 CRI-O 并不是 Docker ，它不支持像 tag、save、load 这些操作，所以这里指定 pause 镜像为自己本地的镜像。

CRI-O 也可以从 Harbor 中拉取镜像，不过会像 docker 一样出现 x509 错误，所以这里将 harbor 镜像仓库的地址加入到了 insecure_registries。

拉取镜像命令（crictl 不支持 login 操作，需要显示指定用户名密码）：

```bash
$ crictl pull registry.prod.bbdops.com/common/kube-apiserver:v1.18.4 --creds username:password
```

> PS: crictl 必须使用 root 用户或 sudo 权限，需要将 /usr/local/bin 目录加入 /etc/sudoers 中的 sudo 执行权限目录。



## kubeadm 高可用部署

配置软件源：

```
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.testing.com/list/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
```

安装：

```bash
$ sudo yum install kubeadm kubelet kubectl -y
```

修改 `/etc/sysconfig/kubelet`中的内容，这一步很关键：

````
KUBELET_EXTRA_ARGS=--feature-gates="AllAlpha=false,RunAsGroup=true" --container-runtime=remote --cgroup-driver=systemd --container-runtime-endpoint='unix:///var/run/crio/crio.sock' --runtime-request-timeout=5m
````

enable 一下 kubelet：

```bash
$ systemctl enable kubelet
```

添加内核模块：

```bash
$ modprobe br_netfilter
```

在 `/etc/sysctl.conf` 中添加内核参数：

```
net.ipv4.ip_forward = 1
vm.swappiness = 0
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
```

使用 `sysctl -p` 使之生效。

使用 `swapoff -a` 关闭交换分区，同时 `/etc/fstab` 中注释掉交换分区。



生成 kubeadm 配置文件：

```bash
$ kubeadm config print init-defaults > kubeadm-config.yaml
```

`/root/kubeadm-config.yaml` 内容如下：

```yaml
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 10.28.63.16
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/crio/crio.sock
  name: kubenode1.prod.bbdops.com
---
apiServer:
  timeoutForControlPlane: 2m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.prod.bbdops.com/common
kind: ClusterConfiguration
controlPlaneEndpoint: 10.28.63.16:6443
kubernetesVersion: v1.18.4
networking:
  dnsDomain: cluster.local
  PodSubnet: 10.85.0.0/16
  serviceSubnet: 10.96.0.0/16
scheduler: {}
```

注意这里的 `PodSubnet` 要和 `/etc/cni/net.d/10-crio-bridge.conf` 中的一致。

`controlPlaneEndpoint` 在需要做高可用时必须要加入，这里最好用一个虚拟 IP 来代理。

使用 `kubeadm config images list` 查看 kubeadm 需要的镜像，我已经把这些镜像放到 `registry.prod.bbdops.com/common` 中了，所以这里 `imageRepository` 改成对应的镜像前缀。

`criSocket` 要配置为 crio 的 socket 文件。



部署：

```bash
$ kubeadm init --config kubeadm-config.yaml --upload-certs
```

如果想卸载，可以执行 `kubeadm reset --cri-socket /var/run/crio/crio.sock`

其他节点作为主控节点加入：

```bash
$ kubeadm join 10.28.63.16:6443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:23868e8ddd7888c412d5579d8d1d3e6ae7678d19e146bbae86106767c2c45add \
    --control-plane --certificate-key 1c67096a3e1938d552eafbc913f8ef7d0ee966b097da21ce0c508603b29540ea
```

也可以作为 work 节点加入：

````bash
$ kubeadm join 10.28.63.16:6443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:23868e8ddd7888c412d5579d8d1d3e6ae7678d19e146bbae86106767c2c45add 
````

这样集群就搭建好了。























