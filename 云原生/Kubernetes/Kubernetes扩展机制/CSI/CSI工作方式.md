

# CSI 工作方式

CSI 是 Kubernetes 中一个比较新的概念，用于对接储存系统。



## 学习资源

CSI 公开课地址：https://edu.aliyun.com/lesson_1651_13092?spm=5176.254948.1387840.13.2c0acad2WRrabG#_13092

CSI 的文档库地址：https://github.com/container-storage-interface/spec

CSI Kubernetes 官方文档：https://kubernetes-csi.github.io/docs/



## Kubernetes 挂载 Volume 过程

下面来梳理一下从一个 PVC 创建，到实际使用的过程。调用过程如下图所示。

![image-20200818210519333](../../../../resource/image-20200818210519333.png)

首先，根据 List - Watch 机制，kubelet、kube-controller-manager、kube-schduler 一直都在监听 kube-apiserver。

1. 创建一个带有 PVC 的 Pod （使用动态储存卷）
2. kube-controller-manager  中的 PV Controller 监听到 kube-apiserver 中有新的处于待绑定状态的 PVC 对象后，会调用 Volume Plugin（in -tree 或 out-of-tree）创建储存卷，并创建与之对应的 PV 对象。
3. Kube-scheduer 根据 Pod 配置，节点状态，PV 配置等信息，把 Pod 调度到某个节点上。
4. Kube-controller-manager 中的 AD Controller 发现 Pod 和 PV 处于待挂载状态，调用 Volume Plugin（in-tree 或 out-of-tree），将设备挂载到目标节点上，比如 /dev/rbd0，
5. 在 Pod 实际运行的节点上，kubelet 等待挂载完成后，通过 Volume Plugin 把 /dev/rbd0 挂载到指定目录，比如： /var/lib/kubelet/pods/532a00db-251d-4569-8ada-984380b05b1b/volumes/kubernetes.io~csi/pvc-cdc47d0c-23b5-495e-911f-037cc031d0d4/mount
6. kubelet 被告知目录挂载完毕后，启动 Pod 中的容器，用 Docker -v (bind) 的方式将已经挂载到本地的卷映射到容器中！





![image-20200318162205853](../../../../resource/image-20200318162205853.png)



