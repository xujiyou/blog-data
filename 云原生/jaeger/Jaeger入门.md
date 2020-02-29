# Jaeger 入门

安装 Istio 后，执行：

```
$ kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=jaeger -o jsonpath='{.items[0].metadata.name}') 15032:16686 &
```

然后就可以通过 `http://localhost:15032`来访问jaeger的UI



不依赖 Istio，下面单独装一个玩玩。

首先下载 jaeger 的二进制文件，下载地址：https://github.com/jaegertracing/jaeger/releases

启动：

```bash
$ jaeger-all-in-one --collector.zipkin.http-port=9411
```

UI 界面在 http://localhost:16686 



## 架构

先来学习下术语。

### Span

一个 Span 是一个逻辑工作单元，其具有操作名称，开始时间和持续时间。 Span 可以嵌套，从而建立因果关系模型。

### Trace

Trace 是一个系统的 数据路径或执行路径，并且可以作为一个有向无环图。



## 组件

Jaeger 可以部署为一个 one in all 二进制程序，也可以作为一个可扩展的分布式系统，