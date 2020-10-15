# Docker 错误汇总

MacOS 执行 `docker login` 是报错（采用 ToolBox 方式安装）：

```
Error saving credentials: error storing credentials - err: exec: "docker-credential-desktop": executable file not found in $PATH, out: ``
```

Fix:

```
$ vim ~/.docker/config.json
```

修改为：

```
"credStore": ""
```



## 证书错误

由于是内网镜像库，证书是自签的，所以本地会出x509错误：

```
x509: certificate signed by unknown authority
```

可以修改 `/etc/docker/daemon.json`，加入以下内容：

```
"insecure-registries": ["registry.prod.bbdops.com"]
```

重启docker后就没有错误了！！！



## 删除docker 日志

在linux上，容器日志一般存放在`/var/lib/docker/containers/container_id/`下面， 以json.log结尾的文件（业务日志）很大。





## 磁盘容量

docker 启动一个容器后默认根分区大小为10GB，通过docker info可以看见默认大小为10G,有时会不够用需要扩展。

```bash
$ docker info | grep Base
```



## driver 错误

错误：

```
time="2020-10-15T15:57:02.691043891+08:00" level=error msg="[graphdriver] prior storage driver overlay2 failed: driver not supported"
Oct 15 15:57:02 ct1.test.bbdops.com dockerd[20860]: Error starting daemon: error initializing graphdriver: driver not supported
```

解决方案，在 `/etc/docker/daemon.json` 中添加：

```json
{
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
```

