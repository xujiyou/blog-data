# 创建 ServiceMonitor

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: zookeeper-cluster-monitor
  namespace: cattle-prometheus
  labels:
    app: zookeeper-cluster
spec:
  namespaceSelector:
    matchNames:
    - drift-test
  selector:
    matchLabels:
      app: zookeeper-cluster
  endpoints:
  - port: metrics
    interval: 10s
```

注意这里 namespace 要和 Prometheus Operator 所在的命名空间一致。port 要写字符串，即 service 中的命名端口。

创建：

```bash
$ kubectl apply -f zookeeper-metrics.yaml
```

创建完成通过：

```bash
$ kubectl logs prometheus-cluster-monitoring-0 prometheus-config-reloader -n cattle-prometheus
```

来查看创建日志，我这里三分钟才创建完成。。。

注意这个日志的时间是 UTC 时间，需要加 8 个小时才是北京时间。

我的日志如下：

```properties
level=info ts=2020-06-09T08:25:46.218151302Z caller=reloader.go:286 msg="Prometheus reload triggered" cfg_in=/etc/prometheus/config/prometheus.yaml.gz cfg_out=/etc/prometheus/config_out/prometheus.e
nv.yaml rule_dirs=

level=info ts=2020-06-09T08:31:46.218111422Z caller=reloader.go:286 msg="Prometheus reload triggered" cfg_in=/etc/prometheus/config/prometheus.yaml.gz cfg_out=/etc/prometheus/config_out/prometheus.e
nv.yaml rule_dirs=

level=info ts=2020-06-09T08:34:46.223448797Z caller=reloader.go:286 msg="Prometheus reload triggered" cfg_in=/etc/prometheus/config/prometheus.yaml.gz cfg_out=/etc/prometheus/config_out/prometheus.e
nv.yaml rule_dirs=
```

通过日志发现，这个容器是三分钟自动刷新一下配置！！！所以会有三分钟延迟。



## 外部服务

有的服务不是 Kubernetes 内部的，但是也需要接入到 Prometheus 内部，需要这样配置：

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: expose-ceph-metrics
  namespace: cattle-prometheus
  labels:
    k8s-app: ceph-metrics
    app: exporter-ceph
    io.cattle.field/appId: cluster-monitoring
    release: cluster-monitoring
subsets:
  - addresses:
      - ip: 10.28.112.11
        nodeName: test-kubenode-1
        targetRef:
          kind: Node
          name: test-kubenode-1
    ports:
      - name: metrics
        port: 9283
        protocol: TCP

---

apiVersion: v1
kind: Service
metadata:
  name: expose-ceph-metrics
  namespace: cattle-prometheus
  labels:
    app: exporter-ceph
    k8s-app: ceph-server
    io.cattle.field/appId: cluster-monitoring
    release: cluster-monitoring
spec:
  clusterIP: None
  type: ClusterIP
  sessionAffinity: None
  ports:
    - name: metrics
      port: 9283
      protocol: TCP
      targetPort: 9283

---

apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: exporter-ceph-cluster-monitoring
  namespace: cattle-prometheus
  labels:
    app: exporter-ceph
    io.cattle.field/appId: cluster-monitoring
    release: cluster-monitoring
    source: rancher-monitoring
spec:
  namespaceSelector:
    matchNames:
      - cattle-prometheus
  selector:
    matchLabels:
      k8s-app: ceph-server
  endpoints:
    - port: metrics
      interval: 10s
      honorLabels: true
      relabelings:
        - action: replace
          regex: (.+)
          replacement: $1
          sourceLabels:
            - __meta_kubernetes_pod_host_ip
          targetLabel: host_ip
        - action: replace
          regex: (.+)
          replacement: $1
          sourceLabels:
            - __meta_kubernetes_pod_node_name
          targetLabel: node
```

