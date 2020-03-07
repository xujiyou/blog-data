# 二进制部署 Flannel

首先去网上下载二进制包，下载地址：https://github.com/coreos/flannel/releases

下载压缩包，解压，将 flanneld 放入 PATH。

编辑 `/usr/lib/systemd/system/flanneld.service`

```
[Unit]
Description=flanneld
After=network.target
Before=docker.service

[Service]
EnvironmentFile=/etc/kube-flannel/flanneld
ExecStart=/usr/bin/flanneld $FLANNEDLD_ARGS
ExecStartPost=/usr/bin/mk-docker-opts.sh -d /run/flannel/docker
Restart=on-failure
RestartSec=5
Type=notify

[Install]
WantedBy=multi-user.target
```

创建配置文件 `/etc/kube-flannel/flanneld`

```
FLANNEDLD_ARGS=" \
--etcd-endpoints=https://fueltank-1:2379,https://fueltank-2:2379,https://fueltank-3:2379 \
--etcd-keyfile=/etc/etcd/cert/etcd/etcd-key.pem \
--etcd-certfile=/etc/etcd/cert/etcd/etcd.pem \
--etcd-cafile=/etc/etcd/cert/etcd/ca.pem \
--iface=eth0 \
"
```

在启动 flanneld 之前，还需要执行以下命令往 etcd 中写入数据：

```bash
$ export ETCDCTL_API=2
$ etcdctl --ca-file=/etc/etcd/cert/etcd/ca.pem --cert-file=/etc/etcd/cert/etcd.pem --key-file=/etc/etcd/cert/etcd-key.pem --endpoints=https://fueltank-1:2379,
https://fueltank-2:2379,https://fueltank-3:2379 set /coreos.com/network/config  '{ "Network": "10.42.0.0/16", "Backend": {"Type": "vxlan"}}'
```

启动：

```
$ sudo systemctl stop docker
$ sudo systemctl enable flanneld
$ sudo systemctl start flanneld
```

/run/flannel/docker 文件内容：

```
DOCKER_OPT_BIP="--bip=10.42.88.1/24"
DOCKER_OPT_IPMASQ="--ip-masq=true"
DOCKER_OPT_MTU="--mtu=1400"
DOCKER_OPTS=" --bip=10.42.88.1/24 --ip-masq=true --mtu=1400"
```



修改docker.service

文件 `/usr/lib/systemd/system/docker.service`

```
[Unit]
After=network-online.target firewalld.service containerd.service flanneld.service
Requires=docker.socket flanneld.service

[Service]
EnvironmentFile=-/run/flannel/docker
ExecStart=/usr/bin/dockerd $DOCKER_OPTS
```

启动 docker：

```
$ sudo systemctl daemon-reload
$ sudo systemctl start docker
```

