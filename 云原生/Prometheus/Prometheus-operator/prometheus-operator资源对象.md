# Prometheus Operator 资源对象

在 Prometheus Operator 中，有以下几种资源对象：

```
alertmanagers 	 monitoring.coreos.com          true         Alertmanager
podmonitors			 monitoring.coreos.com          true         PodMonitor
prometheuses		 monitoring.coreos.com          true         Prometheus
prometheusrules	 monitoring.coreos.com          true         PrometheusRule
servicemonitors  monitoring.coreos.com          true         ServiceMonitor
```

这几个对象很简单。

Prometheus 用于在 Kubernetes 中部署 Prometheus 集群。

PodMonitor 和 ServiceMonitor 分别用于从 Pod 、 Service 中获取 Metrics 数据的，一般 ServiceMonitor 用的比较多。

PrometheusRule 用于定义 Rule。

Alertmanager 用于报警。