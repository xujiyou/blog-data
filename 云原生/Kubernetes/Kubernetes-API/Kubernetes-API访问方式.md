# Kubernetes API 访问方式

官方文档：https://k8smeetup.github.io/docs/tasks/administer-cluster/access-cluster-api/

## 通过 kubectl 访问集群 API

这种方式最常见了，实际使用的也是 REST API，配置好 kubeconfig 文件就可以访问了（默认~/.kube/config）

下面这条命令获取访问配置：

```bash
$ kubectl config view
```

## 直接访问 REST API

Kubectl 会处理 apiserver 的认证和授权。 如果您要用一个 http 客户端， 比如 `curl` 、 `wget` 或者 浏览器直接访问 REST API ， 有多种方法可以通过 apiserver 的认证和授权：

1. 运行 kubectl 的代理模式（推荐）（kubectl proxy）。 比较推荐使用这种方法， 因为这种方式使用了已存储的 apiserver 的位置信息， 并使用自行签发的证书来验证 apiserver 的身份。 使用这种方式不存在中间人攻击。
2. 另外一种可选的方式， 您可以把 apiserver 的位置信息和凭证信息直接提供给http 客户端。 这种方式可以用于那些被代理拒绝的客户端代码。 为了确保不受到中间人攻击， 你需要往您的浏览器里导入根证书。

### 使用 kubectl 代理

通过下面的命令运行 kubectl 在某个模式下， 在该模式下， 它充当了一个反向代理， 起到了处理 apiserver 的定位和认证的作用。

运行命令如下：

```
$ kubectl proxy --port=8080 &
```

访问非常简单：

```bash
$ curl http://localhost:8080/api/
{
  "kind": "APIVersions",
  "versions": [
    "v1"
  ],
  "serverAddressByClientCIDRs": [
    {
      "clientCIDR": "0.0.0.0/0",
      "serverAddress": "fueltank-1:6443"
    }
  ]
}
```

### 不使用代理

如果要避免使用 kubectl 代理的话，可以通过直接传递一个认证 token 给 apiserver， 比如:

```bash
$ APISERVER=$(kubectl config view | grep server | cut -f 2- -d ":" | tr -d " ")
$ TOKEN=$(kubectl get secrets -o jsonpath="{.items[?(@.metadata.annotations['kubernetes\.io/service-account\.name']=='default')].data.token}"|base64 -d)

$ curl $APISERVER/api --header "Authorization: Bearer $TOKEN" --insecure
{
  "kind": "APIVersions",
  "versions": [
    "v1"
  ],
  "serverAddressByClientCIDRs": [
    {
      "clientCIDR": "0.0.0.0/0",
      "serverAddress": "10.0.1.149:443"
    }
  ]
}
```

如何在 chromer 访问 API那，首先第一步： [chrome没有继续前往.md](../../../其他/日常/chrome没有继续前往.md) 

然后第二步，安装 ModHeader 插件，修改 header 即可。



## 通过编程语言访问

通过 swagger.json 文件，可以生成各种语言的客户端库。



## 在 POD 内访问

在 pod 里定位 apiserver 推荐的方式， 是使用 `kubernetes` 的 DNS 名称，这个名称可以解析到一个可以路由到 apiserver 的服务IP.

对 apiserver 的认证推荐的方式是使用一个[服务账号](https://k8smeetup.github.io/docs/user-guide/service-accounts) 凭证. 在 kube-system 命名空间下， 一个 pod关联的服务账号和与服务账号相关的凭证(token) 位于那个 pod 的每个容器的文件系统树之上 ， 具体路径是 `/var/run/secrets/kubernetes.io/serviceaccount/token`.

如果有可用的证书， 那么这个证书会在每个容器的文件系统树下的这个路径 `/var/run/secrets/kubernetes.io/serviceaccount/ca.crt`, 这个证书可以用来验证服务端的 apiserver 证书。

最后的， 可以用于命名空间 API 操作的默认命名空间, 位于一个文件里， 这个文件在每个容器的`/var/run/secrets/kubernetes.io/serviceaccount/namespace` 文件路径下。

