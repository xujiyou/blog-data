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

```

```







