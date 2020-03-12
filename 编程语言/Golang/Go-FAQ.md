# Go FAQ

依赖下载不下来，设置这个参数试试：

```bash
$ export GOPROXY=https://goproxy.io
```

编译 k8s 中的 kube-apiserver 时报错如下：

```
app/server.go:427:70: undefined: "k8s.io/kubernetes/pkg/generated/openapi".GetOpenAPIDefinitions
```

在 k8s 源码的根目录下执行一下下面的命令就好了：

``` 
$ make kube-apiserver
```

