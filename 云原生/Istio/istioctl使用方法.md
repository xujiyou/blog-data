# istioctl 使用方法

官方教程：https://istio.io/zh/docs/reference/commands/istioctl/



## 自动补全

去官方下载 istio 的包，然后在 tools 目录下，有一个名为 `istioctl.bash` 的文件：

```bash
$ cp tools/istioctl.bash ~/.istioctl.bash
$ echo "source ~/.istioctl.bash" >> ~/.bash_profile
$ source ~/.bash_profile 
```



## Flags

#### --context string

要使用的 kubeconfig 中的 context

#### -h, --help

显示帮助

#### -i, --istioNamespace string

istio 的命名空间，默认为 istio-system

#### -c, --kubeconfig string

指定 kubeconfig 文件

#### -n, --namespace string

选择命名空间



## 常用命令

查看概览：

```bash
$ istioctl proxy-status
$ istioctl ps
```



使用 `proxy-config` 或者 `pc` 命令检索代理的配置信息。