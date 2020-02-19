# Secret

要使用 secret，pod 需要引用 secret。Pod 可以用两种方式使用 secret：作为 [volume](https://kubernetes.io/docs/concepts/storage/volumes/) 中的文件被挂载到 pod 中的一个或者多个容器里，或者当 kubelet 为 pod 拉取镜像时使用。

例子：

配置 Pod 使用投射卷作存储：https://kubernetes.io/zh/docs/tasks/configure-pod-container/configure-projected-volume-storage/

从私有仓库拉取镜像：https://kubernetes.io/zh/docs/tasks/configure-pod-container/pull-image-private-registry/

## 内置 secret

#### Service Account 使用 API 凭证自动创建和附加 secret

Kubernetes 自动创建包含访问 API 凭据的 secret，并自动修改您的 pod 以使用此类型的 secret。

如果需要，可以禁用或覆盖自动创建和使用API凭据。但是，如果您需要的只是安全地访问 apiserver，我们推荐这样的工作流程。

参阅 [Service Account](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/) 文档获取关于 Service Account 如何工作的更多信息。