# Kubernetes 调度器性能调优

作为 kubernetes 集群的默认调度器， [kube-scheduler](https://kubernetes.io/zh/docs/concepts/scheduling-eviction/kube-scheduler/#kube-scheduler) 主要负责将 Pod 调度到集群的 Node 上。

在一个集群中，满足一个 Pod 调度请求的所有 Node 称之为 *可调度* Node。 调度器先在集群中找到一个 Pod 的可调度 Node，然后根据一系列函数对这些可调度 Node 打分， 之后选出其中得分最高的 Node 来运行 Pod。 最后，调度器将这个调度决定告知 kube-apiserver，这个过程叫做 *绑定（Binding）*。

这篇文章将会介绍一些在大规模 Kubernetes 集群下调度器性能优化的方式。

在大规模集群中，你可以调节调度器的表现来平衡调度的延迟（新 Pod 快速就位）和精度（调度器很少做出糟糕的放置决策）。

你可以通过设置 kube-scheduler 的 `percentageOfNodesToScore` 来配置这个调优设置。 这个 KubeSchedulerConfiguration 设置决定了调度集群中节点的阈值。



## 设置阈值

`percentageOfNodesToScore` 选项接受从 0 到 100 之间的整数值。0 值比较特殊，表示 kube-scheduler 应该使用其编译后的默认值。 如果你设置 `percentageOfNodesToScore` 的值超过了 100，kube-scheduler 的表现等价于设置值为 100。

要修改这个值，编辑 kube-scheduler 的配置文件（通常是 `/etc/kubernetes/config/kube-scheduler.yaml`），然后重启调度器。

修改完成后，你可以执行

```bash
kubectl get componentstatuses
```

来检查该 kube-scheduler 组件是否健康。输出类似如下：

```
NAME                 STATUS    MESSAGE             ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
...
```



## 节点打分阈值

要提升调度性能，kube-scheduler 可以在找到足够的可调度节点之后停止查找。在大规模集群中，比起考虑每个节点的简单方法相比可以节省时间。

你可以使用整个集群节点总数的百分比作为阈值来指定需要多少节点就足够。 kube-scheduler 会将它转换为节点数的整数值。在调度期间，如果 kube-scheduler 已确认的可调度节点数足以超过了配置的百分比数量， kube-scheduler 将停止继续查找可调度节点并继续进行 [打分阶段](https://kubernetes.io/zh/docs/concepts/scheduling-eviction/kube-scheduler/#kube-scheduler-implementation)。

[调度器如何遍历节点](https://kubernetes.io/zh/docs/concepts/scheduling-eviction/scheduler-perf-tuning/#how-the-scheduler-iterates-over-nodes) 详细介绍了这个过程。

### 默认阈值

如果你不指定阈值，Kubernetes 使用线性公式计算出一个比例，在 100-node 集群下取 50%，在 5000-node 的集群下取 10%。 这个自动设置的参数的最低值是 5%。

这意味着，调度器至少会对集群中 5% 的节点进行打分，除非用户将该参数设置的低于 5。

如果你想让调度器对集群内所有节点进行打分，则将 `percentageOfNodesToScore` 设置为 100。

## 示例

下面就是一个将 `percentageOfNodesToScore` 参数设置为 50% 的例子。

```yaml
apiVersion: kubescheduler.config.k8s.io/v1alpha1
kind: KubeSchedulerConfiguration
algorithmSource:
  provider: DefaultProvider

...

percentageOfNodesToScore: 50
```

### 调节 percentageOfNodesToScore 参数

`percentageOfNodesToScore` 的值必须在 1 到 100 之间，而且其默认值是通过集群的规模计算得来的。 另外，还有一个 50 个 Node 的最小值是硬编码在程序中。

> **说明：**
>
> 当集群中的可调度节点少于 50 个时，调度器仍然会去检查所有的 Node，因为可调度节点太少，不足以停止调度器最初的过滤选择。
>
> 同理，在小规模集群中，如果你将 `percentageOfNodesToScore` 设置为一个较低的值，则没有或者只有很小的效果。
>
> 如果集群只有几百个节点或者更少，请保持这个配置的默认值。改变基本不会对调度器的性能有明显的提升。

值得注意的是，该参数设置后可能会导致只有集群中少数节点被选为可调度节点，很多 node 都没有进入到打分阶段。这样就会造成一种后果，一个本来可以在打分阶段得分很高的 Node 甚至都不能进入打分阶段。

由于这个原因，这个参数不应该被设置成一个很低的值。通常的做法是不会将这个参数的值设置的低于 10。很低的参数值一般在调度器的吞吐量很高且对 node 的打分不重要的情况下才使用。换句话说，只有当你更倾向于在可调度节点中任意选择一个 Node 来运行这个 Pod 时，才使用很低的参数设置。

### 调度器做调度选择的时候如何覆盖所有的 Node

如果你想要理解这一个特性的内部细节，那么请仔细阅读这一章节。

在将 Pod 调度到 Node 上时，为了让集群中所有 Node 都有公平的机会去运行这些 Pod，调度器将会以轮询的方式覆盖全部的 Node。你可以将 Node 列表想象成一个数组。调度器从数组的头部开始筛选可调度节点，依次向后直到可调度节点的数量达到 `percentageOfNodesToScore` 参数的要求。在对下一个 Pod 进行调度的时候，前一个 Pod 调度筛选停止的 Node 列表的位置，将会来作为这次调度筛选 Node 开始的位置。

如果集群中的 Node 在多个区域，那么调度器将从不同的区域中轮询 Node，来确保不同区域的 Node 接受可调度性检查。如下例，考虑两个区域中的六个节点：

```
Zone 1: Node 1, Node 2, Node 3, Node 4
Zone 2: Node 5, Node 6
```

调度器将会按照如下的顺序去评估 Node 的可调度性：

```
Node 1, Node 5, Node 2, Node 6, Node 3, Node 4
```

在评估完所有 Node 后，将会返回到 Node 1，从头开始。









