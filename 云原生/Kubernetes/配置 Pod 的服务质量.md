# 配置 Pod 的服务质量

文档地址：https://kubernetes.io/zh/docs/tasks/configure-pod-container/quality-service-pod/

本文介绍怎样配置 Pod 让其获得特定的服务质量（QoS）类。Kubernetes 使用 QoS 类来决定 Pod 的调度和驱逐策略。

Kubernetes 创建 Pod 时就给它指定了下列一种 QoS 类：

- Guaranteed
- Burstable
- BestEffort

对于 QoS 类为 Guaranteed 的 Pod：

- Pod 中的每个容器必须指定内存请求和内存限制，并且两者要相等。
- Pod 中的每个容器必须指定 CPU 请求和 CPU 限制，并且两者要相等。

如果满足下面条件，将会指定 Pod 的 QoS 类为 Burstable：

- Pod 不符合 Guaranteed QoS 类的标准。
- Pod 中至少一个容器具有内存或 CPU 请求。

对于 QoS 类为 BestEffort 的 Pod，Pod 中的容器必须没有设置内存和 CPU 限制或请求。