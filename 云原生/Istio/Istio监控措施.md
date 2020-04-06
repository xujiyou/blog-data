# Istio 监控措施

Istio 一共有四种可视化监控措施，分别是：Kiali、Jaeger、Grafana、Prometheus。

可以通过在安装时开启，或者使用 `helm upgrade` 命令来启用。

- Kiali：--set kiali.enabled=true
- Jaeger：--set tracing.enabled=true
- Grafana：--set grafana.enabled=true
- Prometheus：--set prometheus.enabled=true

这些可视化界面还是多看，然后知道数据是哪里来的，对于 Grafana 来说，还要会自定义图表。

