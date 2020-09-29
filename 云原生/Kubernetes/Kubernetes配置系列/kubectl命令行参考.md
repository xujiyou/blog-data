# kubectl 命令行参考

概述：https://kubernetes.io/zh/docs/reference/kubectl/overview/

配置介绍：https://kubernetes.io/zh/docs/reference/kubectl/kubectl/

官方文档地址：https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands

port-forward 可以直接映射 Pod 的端口。

```bash
$ kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=jaeger -o jsonpath='{.items[0].metadata.name}') 15032:16686 &
```

查看集群信息：

```bash
$ kubectl cluster-info
```

`kubectl -h` 会列举出所有子命令及其分组。`kubectl options` 会列举出 kubectl 的全局选项。

kubectl 版本为 1.17.3



## 全局选项

一共 33 个全局选项。

- **--add-dir-header=false** 如果为 true 的话，会将文件目录放到 HTTP 请求头中
- **--alsologtostderr=false** 若为 true，则将日志输出到标准错误
- **--as=''** 用于本次操作的用户，比如测试 admin 用户是否有权限：`kubectl auth can-i list endpoints --as admin`
- **--as-group=[]** 作用同上
- **--cache-dir='/home/admin/.kube/http-cache'** 默认的 HTTP 缓存目录
- **--certificate-authority=''** CA文件路径
- **--client-certificate=''** 证书文件路径
- **--client-key=''** 私钥文件路径
- **--cluster=''** kubeconfig 中使用哪个集群
- **--context=''** kubeconfig 中使用哪个 context
- **--insecure-skip-tls-verify=false** 如果为true，将不检查服务器证书的有效性。 这将
  使您的HTTPS连接不安全
- **--kubeconfig=''** kubeconfig 路径，默认为 ~/.kube/config
- **--log-backtrace-at=:0** 当日志在文件行数 N 时，发出堆栈跟踪
- **--log-dir=''** 日志目录，如果不为空，往这个目录写日志
- **--log-file=''** 日志文件
- **--log-file-max-size=1800** 定义日志文件可以增长到的最大大小。 单位为兆字节。 如果值为0，
  文件的最大大小是无限的。
- **--log-flush-frequency=5s** 两次日志刷新之间的最大秒数
- **--logtostderr=true** 错误日志记录到标准错误而不是文件
- **--match-server-version=false** 需要服务器版本以匹配客户端版本
- **-n, --namespace=''** 指定命名空间
- **--password=''** apiserver 的认证密码
- **--profile='none'** 获取配置，可以是：(none|cpu|heap|goroutine|threadcreate|block|mutex)
- **--profile-output='profile.pprof'** profile 写入的文件
- **--request-timeout='0'** 超时时间
- **-s, --server=''** apiserveer 的地址
- **--skip-headers=false** 如果是 true ，避免在日志中出现 header 前缀
- **--skip-log-headers=false** 如果是 true ，在打开日志是跳过 header
- **--stderrthreshold=2** 超过此日志级别，就要重定向到标准错误了。
- **--token=''** token 认证
- **--user=''** kubeconfig 中使用的 user
- **--username=''** 用于 apiserver 认证的用户名
- **--v=0** 日志级别
- **--vmodule=** 以逗号分隔的pattern = N设置列表，用于文件过滤的日志记录



## 子命令

一共有 41 个子命令，分成了 8 组。下面先列一下，再分别来学一下。

1. **Basic Commands (Beginner)**

   **create** 根据文件或标准输入来创建资源

   **expose** 用于创建 service，公开端口

   **run** 在集群中运行指定的镜像

   **set** 为资源对象设置一些东西

2. **Basic Commands (Intermediate)**

   **explain** 查看资源对象的Documentation

   **get** 展示一个或多个资源列表

   **edit** 编辑一个资源对象

   **delete** 通过文件，标准输入，资源名或 label 来删除资源对象

3. **Deploy Commands**

   **rollout** 对资源进行管理，如回滚，查看状态，暂停资源等

   **scale** 扩展资源对象的数量

   **autoscale** 自动扩展

4. **Cluster Management Commands**

   **certificate** 修改资源证书

   **cluster-info** 显示集群信息

   **top** 显示 (CPU/Memory/Storage) 使用情况

   **cordon** 将节点标记为不可调度

   **uncordon** 将节点标记为可调度

   **drain**  准备维护节点

   **taint** 更新一个或多个节点的 taints

5. **Troubleshooting and Debugging Commands**

   **describe** 展示资源的详细信息

   **logs** 获取容器日志

   **attach** 连接到一个运行整的容器上

   **exec** 在运行中的运行里面运行命令

   **port-forward** 将 Pod 中的端口映射到本机

   **proxy** 创建一个 kube-apiserver 的代理

   **cp** 在主机和容器之间来回拷贝文件或目录

   **auth** 用于检查权限

6. **Advanced Commands**

   **diff** 显示想要更新版本的变化

   **apply** 可自动创建或更新资源对象

   **patch** 为资源对象添加属性

   **replace** 替换资源对象

   **wait** 实验性：等待一种或多种资源的特定条件。

   **convert** 转换两个不同版本的配置

   **kustomize** 从目录或远程URL构建Kustomization目标。

7. **Other Commands**

   **api-resources** 查看 API 资源有哪些

   **api-versions** 查看 API 版本有哪些

   **config** 修改 kubeconfig 文件

   **plugin** 提供用于与插件交互的实用程序。

   **version** 查看版本



## 常用命令

将 PV 从 release 状态变为 avaliveable：

````bash
$ kubectl patch pv pvc-7615872a-6328-46f3-9d3a-a636db7c0975 -p '{"spec":{"claimRef": null}}'
````

查看 node 的 labels：

```bash
$ kubectl get node --show-labels
```



## 自动补全

```bash
$ yum install bash-completion -y
$ source /usr/share/bash-completion/bash_completion
$ source <(kubectl completion bash)
$ echo "source <(kubectl completion bash)" >> ~/.bashrc
```



## 给 node 打标签

```
kubectl label nodes node1 key1=val1
```









