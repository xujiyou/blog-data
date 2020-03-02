# Prometheus 可视化

Prometheus 自带了一个可视化的UI组件，地址在 ：http://localhost:9090/graph

在这个 UI 组件有一个 PromQL 的执行端口，另外还可以看到 告警 和 Prometheus 自身的一些信息。

这个 UI 组件 主要用于临时查询和调试，如果用于展示高大上的监控界面，需要使用  [Grafana](https://prometheus.io/docs/visualization/grafana/) 或 [控制台模板](https://prometheus.io/docs/visualization/consoles/)。

## 控制台模板

控制台模板允许使用[Go模板语言](https://golang.org/pkg/text/template/)创建任意控制台。这些是从Prometheus服务器提供的。

使用 http://localhost:9090/consoles/node.html 就可以访问 Prometheus 包下 consoles 目录中的 html 文件了。

这种方式可以高度自定义，从 html 中可以获取到 Prometheus 中的数据，不过太麻烦了，不推荐。

## Grafana

Grafana 的图形界面非常炫酷，但是怎样使用还要仔细学习一下 Grafana。就像学习 Kibana 一样。