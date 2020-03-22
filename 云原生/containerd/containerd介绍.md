# containerd 介绍

最新版本的 docker 都是通过 containerd 来管理镜像和容器的。

containerd 也是 docker 提取开源并送给 CNCF 的开源项目。

kubernetes 也可以直接使用 containerd。



containerd 官网：https://containerd.io/

containerd 官网的文档比较简单，只有一个 **overview** 和 **Getting started**



更多文档在：https://github.com/containerd/containerd/tree/master/docs



## Overview

containerd 的各种库放在了GitHub中：

[containerd/containerd](https://github.com/containerd/containerd) 主要的代码库，包含容器运行时。

[containerd/containerd.io](https://github.com/containerd/containerd.io) containerd 的官方网站

[containerd/cri](https://github.com/containerd/cri) containerd 的 CRI 插件

[containerd/project](https://github.com/containerd/project) 跨容器存储库使用的实用程序，例如脚本，通用文件和核心文档

[containerd/ttrpc](https://github.com/containerd/ttrpc) 容器使用的gRPC版本（专为低内存环境设计）



## **Getting started**

安装完 docker 后，系统中已经有 containerd 了，下面来写代码：

```go
package main

import (
	"context"
	"github.com/containerd/containerd"
	"github.com/containerd/containerd/namespaces"
	"log"
)

func main() {
	client, err := containerd.New("/run/containerd/containerd.sock")
	if err != nil {
		log.Fatalln(err)
	}

	defer client.Close()
	ctx := namespaces.WithNamespace(context.Background(), "example")
	image, err := client.Pull(ctx, "docker.io/library/redis:alpine", containerd.WithPullUnpack)
	if err != nil {
		log.Fatalln(err)
	}
	log.Printf("Successfully pulled %s image\n", image.Name())
}

```

注意因为 containerd 还不支持 go modules，所以应该把代码放到 $GOPATH/src 目录下才行。

编译：

```
$ go build
```

执行：

```
$ sudo ./containerd-test
2020/03/22 20:57:08 Successfully pulled docker.io/library/redis:alpine image
```

可以了，测试通过。



