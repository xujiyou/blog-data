# Istio 安装教程

Istio 1.5 发布了，架构发生了改变，幸好之前没深入学习，这下可以尝鲜了。

官方安装教程(istioctl 方式)：https://istio.io/docs/setup/install/istioctl/

首先去官方下载二进制包，或者用脚本下载

```
curl -L https://istio.io/downloadIstio | sh -
```

下载完成后，会得到一个 `istio-1.4.2` 目录，里面包含了：

- `install` : 安装文件
- `samples` : 示例应用
- `bin` : 包含 istioctl 二进制文件，可以复制到 `/usr/bin` 来使用
- `tools`: 一些脚本文件

将 istioctl 拷贝到 `/usr/bin/` 中：

```
$ cp bin/istioctl /usr/local/bin/
```



### 开启 istioctl 的自动补全功能

bash：

```
$ cp tools/istioctl.bash ~/
```

在 `~/.bashrc` 中添加一行：

```
source ~/istioctl.bash
```

应用生效：

```
$ source ~/.bashrc
```

zsh:

将 `tools` 目录中的 `_istioctl` 拷贝到 $HOME 目录中：

```
$ cp tools/_istioctl ~/
```

在 `~/.zshrc` 中添加一行：

```
source ~/_istioctl
```

应用生效：

```
$ source ~/.zshrc
```

istioctl 提供了多种安装配置文件，可以通过下面的命令查看：

```
$ istioctl profile list
Istio configuration profiles:
    sds
    default
    demo
    minimal
    remote
```

它们之间的差异如下：

|                        | default | demo  | minimal | sds   | remote |
| :--------------------: | :------ | :---- | :------ | :---- | :----- |
|      **核心组件**      |         |       |         |       |        |
|     istio-citadel      | **X**   | **X** |         | **X** | **X**  |
|  istio-egressgateway   |         | **X** |         |       |        |
|      istio-galley      | **X**   | **X** |         | **X** |        |
|  istio-ingressgateway  | **X**   | **X** |         | **X** |        |
|    istio-nodeagent     |         |       |         | **X** |        |
|      istio-pilot       | **X**   | **X** | **X**   | **X** |        |
|      istio-policy      | **X**   | **X** |         | **X** |        |
| istio-sidecar-injector | **X**   | **X** |         | **X** | **X**  |
|    istio-telemetry     | **X**   | **X** |         | **X** |        |
|      **附加组件**      |         |       |         |       |        |
|        Grafana         |         | **X** |         |       |        |
|     istio-tracing      |         | **X** |         |       |        |
|         kiali          |         | **X** |         |       |        |
|       prometheus       | **X**   | **X** |         | **X** |        |

其中标记 **X** 表示该安装该组件。

如果只是想快速试用并体验完整的功能，可以直接使用配置文件 `demo` 来部署

接下来正式安装 istio，命令如下：

```bash
$ istioctl manifest apply --set profile=demo \
   --set cni.enabled=true --set cni.components.cni.namespace=kube-system \
   --set values.gateways.istio-ingressgateway.type=ClusterIP --set security.enabled=false
```

部署完成后，查看各组件状态：

```bash
$ kubectl -n istio-system get pod
$ kubectl -n kube-system get pod -l k8s-app=istio-cni-node
```

暴露 Dashboard：

```bash
$ istioctl dashboard --help
```



## 验证安装

```bash
$ istioctl manifest generate <your original installation options> > $HOME/generated-manifest.yaml
$ istioctl verify-install -f $HOME/generated-manifest.yaml
```



## 卸载：

```bash
$ istioctl manifest generate --set profile=demo | kubectl delete -f -
```

