# kube-controller-manager 内置的控制器

kube-controller-manager 是 Kubernetes 的大脑，控制着 apiserver 中的各种资源。

kube-controller-manager 内包含很多内置的控制器，在  Kube-controller-manager 的 `--controllers` 配置中可以选择要使用的控制器，默认是全部。下面依次了解各种控制器的作用。

#### --controllers strings

开启控制器的列表，* 意味着启用所有默认控制器，“foo” 表示开启名字为 foo 的控制器，“-foo” 表示关闭名字为 foo 的控制器。

所有默认开启的控制器，共36个：

attachdetach, bootstrapsigner, cloud-node-lifecycle, clusterrole-aggregation, cronjob, csrapproving, csrcleaner, csrsigning, daemonset, deployment, disruption, endpoint, endpointslice, garbagecollector, horizontalpodautoscaling, job, namespace, nodeipam, nodelifecycle, persistentvolume-binder, persistentvolume-expander, podgc, pv-protection, pvc-protection, replicaset, replicationcontroller, resourcequota, root-ca-cert-publisher, route, service, serviceaccount, serviceaccount-token, statefulset, tokencleaner, ttl, ttl-after-finished。

默认关闭的控制器：

 bootstrapsigner, tokencleaner

默认值为 [*]



## attachdetach 控制器

