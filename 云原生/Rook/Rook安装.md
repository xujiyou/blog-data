# Rook 安装

安装前提：https://rook.io/docs/rook/v1.2/flexvolume.html

这里有坑，要保证相应的目录有读写权限

Ceph 版安装文档：https://rook.io/docs/rook/v1.2/ceph-quickstart.html

ceph 的本地储存地址可以自定义。

另外，还要注意 operator.yaml 中 kubelet 的储存地址要和 kubelet 服务的储存地址一致！！！

集群删除时，要手动把 rook-ceph 命名空间删掉。

