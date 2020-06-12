# Docker 储存驱动

查看当前系统中 Docker 使用的储存驱动：

```bash
$ docker info | grep "Storage Driver"
Storage Driver: overlay2
```

参考：https://zhuanlan.zhihu.com/p/41958018

当 pull 一个镜像时：

```
[root@drift-1 ~]# docker pull ubuntu
Using default tag: latest
latest: Pulling from library/ubuntu
d51af753c3d3: Pull complete 
fc878cd0a91c: Pull complete 
6154df8ff988: Pull complete 
fee5db0ff82f: Pull complete 
Digest: sha256:747d2dbbaaee995098c9792d99bd333c6783ce56150d1b11e333bbceed5c54d7
Status: Downloaded newer image for ubuntu:latest
docker.io/library/ubuntu:latest
```

可以看到镜像是分层的，并且每一层都有一个签名。这个 ubuntu 镜像一共 5 层，这5层都是只读的。

起一个容器：

```bash
$ docker run -it ubuntu /bin/bash
```

查看挂载点：

![image-20200612181319839](../../resource/image-20200612181319839.png)

这里可以看到，lowerdir  有五个目录，upperdir 一个目录，workdir 一个目录，这七个目录合并在一起组成了挂在了根目录。

其中`lowerdir`是镜像只读层，`upperdir`是容器可读可写层，`workdir`是执行涉及修改`lowerdir`执行`copy_up`操作的中转层。

这些目录的实际内容如下：

![image-20200612184652016](../../resource/image-20200612184652016.png)

下面来验证一下：

在容器内执行：

```bash
$ echo "hello world" > /abc.txt
```

然后看 upperdir 中的文件：

![image-20200612185312608](../../resource/image-20200612185312608.png)

不出所料。并且在 ../merged 文件夹内也有一个完整的目录结构：

![image-20200612185735830](../../resource/image-20200612185735830.png)

这个 merged 文件夹就是容器用到的文件系统。



