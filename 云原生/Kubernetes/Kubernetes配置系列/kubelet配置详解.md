# kubelet 配置详解

官方文档：https://kubernetes.io/zh/docs/reference/command-line-tools-reference/kubelet/

大约 178 个配置项，是各组件中配置项最多的了。

kubelet 的配置比较乱。没有分组，按字符顺序排序的。

这些配置里边好多过期的。

官方推荐使用 --config 参数指定配置文件，可以参考：https://kubernetes.io/zh/docs/tasks/administer-cluster/kubelet-config-file/

动态配置处于 beta 版本了，应该快 release 了。可以通过 yaml 文件来配置组件，并将配置保存在集群中！！！ 