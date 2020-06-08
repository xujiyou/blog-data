# Kubernetes Operator 教程

Operator 特别适合部署有状态的应用，是 CoreOS 发明的。

Operator Hub：https://operatorhub.io/

里面有很多现成的 Operator ，可以拿来直接使用。

官方的入门教程：https://github.com/operator-framework/getting-started

关于 Helm 和 Operator 的对比：https://blog.51cto.com/12462495/2084517

下面把我的一次操作过程记录一下。

## 创建项目

首先去 github 下载个 operator-sdk 的二进制文件，下载地址：https://github.com/operator-framework/operator-sdk

下载完成后放到 Path 里面即可，然后创建项目：

```bash
$ mkdir -p $GOPATH/src/github.com/example-inc/
$ cd $GOPATH/src/github.com/example-inc/
$ export GO111MODULE=on
$ operator-sdk new memcached-operator
$ cd memcached-operator
```

然后获取依赖：

```bash
$ go mod tidy
```

获取完成后，可以使用 Goland 打开 GOPATH 这个目录，不要直接打开  memcached-operator 这个目录，要不 Goland 会下载半天依赖，不知道为嘛这样，感觉这个 GOPATH 坑的一逼。



### 添加 CRD

使用命令来生成 CRD：

```bash
$ operator-sdk add api --api-version=cache.example.com/v1alpha1 --kind=Memcached
```

这会在 pkg/apis/cache/v1alpha1/ 底下生成几个 go 源码文件。

修改 pkg/apis/cache/v1alpha1/memcached_types.go 中的代码：

```go
type MemcachedSpec struct {
    // Size is the size of the memcached deployment
    Size int32 `json:"size"`
}
type MemcachedStatus struct {
    // Nodes are the names of the memcached pods 
    Nodes []string `json:"nodes"`
}
```

然后使用以下命令来生成 CRD：

```bash
$ operator-sdk generate k8s
```

这条命令会在 deploy/crds 中创建一个 CRD 文件 和 一个 Memcached 资源对象文件。



## 添加 Controller

有了 CRD 之后，还要有 CRD Controller，来监听对 CRD 的操作。

生成一个 Controller ：

```bash
$ operator-sdk add controller --api-version=cache.example.com/v1alpha1 --kind=Memcached
```

修改 pkg/controller/memcached/memcached_controller.go ，用这个[`memcached_controller.go`](https://github.com/operator-framework/operator-sdk/blob/master/example/memcached-operator/memcached_controller.go.tmpl) 文件代码来替换其中的代码



## 部署

先部署 CRD：

```bash
$ kubectl create -f deploy/crds/cache.example.com_memcacheds_crd.yaml
```

部署完 CRD 后，可以打 Docker 镜像，在上传，再部署另外几个 yaml 文件：(这步我没操作)

```bash
$ operator-sdk build quay.io/<user>/memcached-operator:v0.0.1
$ sed -i 's|REPLACE_IMAGE|quay.io/<user>/memcached-operator:v0.0.1|g' deploy/operator.yaml
$ docker push quay.io/<user>/memcached-operator:v0.0.1
```

部署 yaml 文件：（这步也没操作）

```bash
$ kubectl create -f deploy/service_account.yaml
$ kubectl create -f deploy/role.yaml
$ kubectl create -f deploy/role_binding.yaml
$ kubectl create -f deploy/operator.yaml
```

上面两步都没进行操作，为嘛那？

因为上面那个 `operator-sdk build` 需要用到本地的 Docker 环境，不能用远程的 Docker 环境！！！我 Mac 上没 Docker 环境（不想装，占资源），遂放弃。

怎么办那，上边安装的是正式环境，也可以用测试环境：

```bash
$ operator-sdk run --local --namespace=default
INFO[0000] Running the operator locally in namespace default.
{"level":"info","ts":1582808377.805665,"logger":"cmd","msg":"Operator Version: 0.0.1"}
{"level":"info","ts":1582808377.805763,"logger":"cmd","msg":"Go Version: go1.14"}
{"level":"info","ts":1582808377.805769,"logger":"cmd","msg":"Go OS/Arch: darwin/amd64"}
{"level":"info","ts":1582808377.805774,"logger":"cmd","msg":"Version of operator-sdk: v0.15.2"}
{"level":"info","ts":1582808377.808009,"logger":"leader","msg":"Trying to become the leader."}
{"level":"info","ts":1582808377.8080451,"logger":"leader","msg":"Skipping leader election; not running in a cluster."}
{"level":"info","ts":1582808388.650317,"logger":"controller-runtime.metrics","msg":"metrics server is starting to listen","addr":"0.0.0.0:8383"}
{"level":"info","ts":1582808388.650487,"logger":"cmd","msg":"Registering Components."}
{"level":"info","ts":1582808388.6507149,"logger":"cmd","msg":"Skipping CR metrics server creation; not running in a cluster."}
{"level":"info","ts":1582808388.650736,"logger":"cmd","msg":"Starting the Cmd."}
```

这样就不用打 Docker 镜像，也不用部署这四个 yaml 文件了。

然后部署上边的那个 Memcached 资源对象文件：

```bash
$ kubectl apply -f deploy/crds/cache.example.com_v1alpha1_memcached_cr.yaml
```

部署完成后，查看效果：

```bash
[admin@fueltank-1 ~]$ kubectl get deployment
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
example-memcached   0/3     3            0           8s
[admin@fueltank-1 ~]$ kubectl get pods
NAME                                 READY   STATUS              RESTARTS   AGE
example-memcached-7c4df9b7b4-8wpx8   0/1     ContainerCreating   0          20s
example-memcached-7c4df9b7b4-k6v4c   0/1     ContainerCreating   0          20s
example-memcached-7c4df9b7b4-prmms   0/1     ContainerCreating   0          20s
[admin@fueltank-1 ~]$ kubectl get memcached
NAME                AGE
example-memcached   66s
[admin@fueltank-1 ~]$ kubectl get memcached example-memcached
NAME                AGE
example-memcached   83s
[admin@fueltank-1 ~]$ kubectl get memcached example-memcached -o yaml
apiVersion: cache.example.com/v1alpha1
kind: Memcached
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"cache.example.com/v1alpha1","kind":"Memcached","metadata":{"annotations":{},"name":"example-memcached","namespace":"default"},"spec":{"size":3}}
  creationTimestamp: "2020-02-27T13:03:43Z"
  generation: 1
  name: example-memcached
  namespace: default
  resourceVersion: "9124926"
  selfLink: /apis/cache.example.com/v1alpha1/namespaces/default/memcacheds/example-memcached
  uid: 6ee9eb3b-51a5-412e-a7eb-8f5bfb567317
spec:
  size: 3
status:
  nodes:
  - example-memcached-7c4df9b7b4-k6v4c
  - example-memcached-7c4df9b7b4-prmms
  - example-memcached-7c4df9b7b4-8wpx8
```

稍微等一会， Pod 就创建好了。

下面可以通过修改 Memcached 资源对象文件来达到修改 Pod 数量的目的：

```yaml
apiVersion: cache.example.com/v1alpha1
kind: Memcached
metadata:
  name: example-memcached
spec:
  # Add fields here
  size: 4
```

 把 3 改成 4。然后重新 apply：

```bash
$ kubectl apply -f deploy/crds/cache.example.com_v1alpha1_memcached_cr.yaml
```

再查看效果，Pod 变成 4 个了，这就达到了通过修改资源文件来操作 Pod 的目的。

```bash
[admin@fueltank-1 ~]$ kubectl get pods
NAME                                 READY   STATUS    RESTARTS   AGE
example-memcached-7c4df9b7b4-8jdkd   1/1     Running   0          25m
example-memcached-7c4df9b7b4-8wpx8   1/1     Running   0          28m
example-memcached-7c4df9b7b4-k6v4c   1/1     Running   0          28m
example-memcached-7c4df9b7b4-prmms   1/1     Running   0          28m
[admin@fueltank-1 ~]$ kubectl get deployment
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
example-memcached   4/4     4            4           29m
```



OK，至此，创建 Operator 的套路就掌握了，剩下的就是研究 golang 代码怎么写了。



## 关于 operator 的命名空间

https://access.redhat.com/documentation/zh-cn/openshift_container_platform/4.2/html/operators/operator-sdk

默认只会监听 operator 所在的命名空间，可以通过在 main.go 中设置：

```go
  options := manager.Options{
		Namespace:          "",
		MetricsBindAddress: fmt.Sprintf("%s:%d", metricsHost, metricsPort),
	}

	mgr, err := manager.New(cfg, options)
```

来监听所有命名空间。