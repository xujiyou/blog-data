# Kubernetes API 相关

在 kubernetes 源码中，有一个 swagger.json 的文件，源码下载地址：https://github.com/kubernetes/kubernetes

![image-20200312105004978](../../resource/image-20200312105004978.png)

然后下载 swagger 二进制文件，下载地址：https://github.com/go-swagger/go-swagger/releases

然后运行：

```bash
$ swagger serve swagger.json 
```

这样就可以看到 k8s 的全部 API 了。

验证 json 文件：

```bash
$ swagger validate swagger.json
```

生成 server 端代码：

```bash
$ swagger generate server -f swagger.json -A one
```





