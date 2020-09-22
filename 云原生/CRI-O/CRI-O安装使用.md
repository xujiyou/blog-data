# CRI-O 安装使用

准备在 CentOS 8 中使用 CRI-O

在 Github 中下载 CRI-O 的二进制压缩包：https://storage.googleapis.com/k8s-conform-cri-o/artifacts/crio-v1.19.0.tar.gz

下载完成后进行解压。



## 安装

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

如果要卸载，需要执行：

```bash
$ sudo make uninstall
```

测试：

```bash
$ sudo su -
$ export PATH=$PATH:/usr/local/bin
$ crictl ps
```





## 运行一个容器

`pod-config.json` 内容:

```

```











