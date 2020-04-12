# operator sdk 教程

关于学习 operator 的细节并不容易学习。

这里先学习一下 [operator-sdk](https://github.com/operator-framework/operator-sdk) 的使用

Github 教程：https://github.com/operator-framework/operator-sdk/blob/master/doc/user-guide.md

首先安装，MacOS安装很容易：

```
$ brew install operator-sdk
```

前提：

- git
- go
- [mercurial](https://www.mercurial-scm.org/downloads) 通过  `brew install mercurial` 安装
- docker
- kubectl

## 创建一个新项目

```
$ mkdir -p $HOME/projects
$ cd $HOME/projects
$ operator-sdk new memcached-operator --repo=github.com/example-inc/memcached-operator
$ cd memcached-operator
```



### 目录结构

文档：https://github.com/operator-framework/operator-sdk/blob/master/doc/project_layout.md

| File/Folders   | Purpose                                                      |
| -------------- | ------------------------------------------------------------ |
| cmd            | 包含 `manager/main.go`，这个文件是 operator 的入口程序. 他会在 pkg/apis/... 低下创建 CRD ，在 pkg/controllers/... 底下创建 控制器 |
| pkg/apis       | 包含了 CRD 的 api. 用户需要编辑 `pkg/apis/<group>/<version>/<kind>_types.go` 文件来定义每种资源类型的API，并将这些包导入其控制器中以监视这些资源类型。 |
| pkg/controller | 这个包包含了控制器的实现. 用户需要编辑 `pkg/controller/<kind>/<kind>_controller.go` 来定义控制器的协调逻辑，以处理指定“kind”的资源类型。 |
| build          | 包含了 `Dockerfile` 和构建脚本来创建 operator                |
| deploy         | 包含了 CRD 和 RBAC 的YAML , 还有部署 opertator 的 YAML 文件  |
| go.mod go.sum  | Go Mudules 用到的依赖文件                                    |



## 管理

main.go 用于初始化和运行 [Manager](https://godoc.org/github.com/kubernetes-sigs/controller-runtime/pkg/manager#Manager).

Manager 可以自动注册 `pkg/apis/...` 目录下的 CRD，也可以运行 `pkg/controller/...` 目录下的控制器。

Manager 可以限制控制器和他监控资源的命名空间。

```go
mgr, err := manager.New(cfg, manager.Options{Namespace: namespace})
```

如果要监听所有命名空间，可以将命名空间留空：

```go
mgr, err := manager.New(cfg, manager.Options{Namespace: ""})
```

也可以使用 [MultiNamespacedCacheBuilder](https://godoc.org/github.com/kubernetes-sigs/controller-runtime/pkg/cache#MultiNamespacedCacheBuilder)  指定一组命名空间：

```go
var namespaces []string // List of Namespaces
// Create a new Cmd to provide shared dependencies and start components
mgr, err := manager.New(cfg, manager.Options{
   NewCache: cache.MultiNamespacedCacheBuilder(namespaces),
   MapperProvider:     restmapper.NewDynamicRESTMapper,
   MetricsBindAddress: fmt.Sprintf("%s:%d", metricsHost, metricsPort),
})
```

main.go 将使用deploy / operator.yaml中定义的 WATCH_NAMESPACE env 值设置管理器的名称空间。

## 添加 CRD

添加一个 APIVersion 为 `cache.example.com/v1alpha1` ，Kind 类型为 `Memcached` 的 CRD：

```bash
$ operator-sdk add api --api-version=cache.example.com/v1alpha1 --kind=Memcached
```

这句命令将会把 CRD Yaml 文件放到 `pkg/apis/cache/v1alpha1/` 目录下。

## 定义 spec 和 status

修改名为 Memcached 的CR的 spec 和 status，在 `pkg/apis/cache/v1alpha1/memcached_types.go` 中

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

修改完成后，生成资源：

```
$ operator-sdk generate k8s
```

生成 CRD 属性字段：

```
$ operator-sdk generate crds
```



## 添加一个控制器

添加一个控制器，用来监听和控制资源：

```
$ operator-sdk add controller --api-version=cache.example.com/v1alpha1 --kind=Memcached
```

在这个例子中 控制器要做以下事情：

- 创建一个 memcached Deployment 
- 确保 Deployment 中 Pod 的数量跟 Memcached 中定义的一致。
- 更新 Memcached CRD 的 status

在 `pkg/controller/memcached/memcached_controller.go` 中可以看到控制器怎么 watch 资源。

