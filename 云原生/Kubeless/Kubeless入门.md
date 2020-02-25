# Kubeless 入门

首先，需要安装，官方文档：https://kubeless.io/docs/quick-start/

这里推荐安装带 **RBAC** 的版本，因为不带 **RBAC** 的版本会有问题。

安装步骤：

```bash
$ kubectl create ns kubeless
$ kubectl create -f https://github.com/kubeless/kubeless/releases/download/v1.0.6/kubeless-v1.0.6.yaml 
```

然后下载 kubeless 命令行，并将其放置到 PATH 路径中（linux，macOS，windows都可安装，使用的是 Kubernetes 的配置文件）。

验证创建是否完成：

```bash
$ $ kubectl get pods -n kubeless
```

待所有 POD 都是 running 之后就可以了。

## Hello world

首先创建 test.py ：

```python
def hello(event, context):
  print event
  return event['data']
```

创建函数：

```bash
$ kubeless function deploy hello --runtime python2.7 \
                                --from-file test.py \
                                --handler test.hello
```

验证：

```bash
$ kubectl get functions
$ kubeless function ls
```

还需验证：

```bash
$ kubectl get pods
```

待 POD 创建完成才行，在墙内，下载速度挺慢的。

待 POD 完成后，验证：

```bash
$ kubeless function call hello --data 'Hello world!'
Hello world!
```

或者：

```bash
$ kubectl proxy -p 8080 &
$ curl -L --data '{"Another": "Echo"}' \
  --header "Content-Type:application/json" \
  localhost:8080/api/v1/namespaces/default/services/hello:http-function-port/proxy/
{"Another": "Echo"}
```

