# Docker 镜像相关

手动下载镜像：

```
$ sudo proxychains4 docker pull quay.io/kiali/kiali:v1.9
```

proxychains4 是代理。

镜像下载完成之后，需要复制到其他系统中，先执行以下命令保存：

```
$ sudo docker save quay.io/kiali/kiali:v1.9 -o  ./kiali1.9.tar
```

再到其他系统中恢复：

```
$ sudo docker load < kiali1.9.tar
```



## 阿里云代理

```bash
sudo vim /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": ["https://4vra6qzb.mirror.aliyuncs.com"]
}
EOF
```

重启docker