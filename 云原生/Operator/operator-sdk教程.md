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
| cmd            | 包含 `manager/main.go`，这个文件是 operator 的入口程序. This instantiates a new manager which registers all custom resource definitions under `pkg/apis/...` and starts all controllers under `pkg/controllers/...` . |
| pkg/apis       | Contains the directory tree that defines the APIs of the Custom Resource Definitions(CRD). Users are expected to edit the `pkg/apis///_types.go` files to define the API for each resource type and import these packages in their controllers to watch for these resource types. |
| pkg/controller | This pkg contains the controller implementations. Users are expected to edit the `pkg/controller//_controller.go` to define the controller's reconcile logic for handling a resource type of the specified `kind`. |
| build          | Contains the `Dockerfile` and build scripts used to build the operator. |
| deploy         | Contains various YAML manifests for registering CRDs, setting up [RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/), and deploying the operator as a Deployment. |
| go.mod go.sum  | The [Go mod](https://github.com/golang/go/wiki/Modules) manifests that describe the external dependencies of this operator. |
| vendor         | The golang [vendor](https://golang.org/cmd/go/#hdr-Vendor_Directories) directory that contains local copies of external dependencies that satisfy Go imports in this project. [Go modules](https://github.com/golang/go/wiki/Modules) manages the vendor directory directly. This directory will not exist unless the project is initialized with the `--vendor` flag, or `go mod vendor` is run in the project root. |