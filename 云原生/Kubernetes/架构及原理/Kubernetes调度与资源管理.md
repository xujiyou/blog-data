# Kubernetes 调度与资源管理

公开课地址：https://edu.aliyun.com/lesson_1651_13090?spm=5176.10731542.0.0.33d720beJkvduZ#_13090



## Kubernetes 调度过程

![image-20201110201509555](../../../resource/image-20201110201509555.png)

调度过程 = 把 Pod 放到合适的 Node 上。

那怎么定义合适？

- 满足 Pod 资源要求
- 满足 Pod 特殊关系要求
- 满足 Node 限制条件的要求
- 集群资源的合理利用



## Kubernetes 基础调度能力

- 资源调度 - 满足 Pod 资源要求
  - Resources: CPU/Memory/Storage/GPU/FGPA
  - QoS: Guaranteed/Burstable/BestEffot
  - Resource Quota
- 关系调度 - 满足 Pod/Node 的特殊关系/条件要求
  - PodAffinity/PodAntiAffinity: Pod 与 Pod 之间的关系
  - NodeSelector/NodeAffinity: 由 Pod 决定适合自己的 Node
  - Taint/Tolerations: 限制调度到某些 Node



#### 资源配置方法

Pod 的资源类型：

- CPU
- Memory
- ephemeral-storage
- Extended-resource: nvmdia.com/gpu

怎么调度：









## Kubernetes 高级调度能力

PodPriority/Preemtion 优先级和抢占

