# Elasticsearch 的 Operator

Elasticsearch 官方推出了一个 operator，名为 ECK，挺好用的。

通过 Elasticsearch Operator 来部署，参见：https://www.elastic.co/cn/downloads/elastic-cloud-kubernetes

通过一个 Elasticsearch Operator 可以部署多个版本的 ES 集群，和多个版本的 Kibana。



## 部署实例

ES：

```yaml
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: elasticsearch-7-6
  namespace: default
spec:
  version: 7.6.2
  http:
    tls:
      selfSignedCertificate:
        disabled: true
  nodeSets:
    - name: default
      count: 3
      config:
        node.master: true
        node.data: true
        node.ingest: true
        node.store.allow_mmap: false
      podTemplate:
        spec:
          initContainers:
            - name: install-plugins-ik
              command:
                - sh
                - -c
                - |
                  bin/elasticsearch-plugin list | grep analysis-ik
                  [[ $? -ne 0 ]] && bin/elasticsearch-plugin install --batch http://mirrors.bbdops.com/list/public/elasticsearch/elasticsearch-analysis-ik-7.6.2.zip
          containers:
            - name: elasticsearch
              env:
                - name: ES_JAVA_OPTS
                  value: -Xms4g -Xmx4g
              resources:
                requests:
                  memory: 4Gi
                  cpu: 0.2
                limits:
                  memory: 8Gi
                  cpu: 2
      volumeClaimTemplates:
        - metadata:
            name: elasticsearch-data
          spec:
            accessModes:
              - ReadWriteOnce
            resources:
              requests:
                storage: 100Gi
            storageClassName: ceph-storage-class
```

注意其中的插件安装方式，这也是坑之一：https://github.com/elastic/elasticsearch/issues/55443

Kibana：

```yaml
apiVersion: kibana.k8s.elastic.co/v1
kind: Kibana
metadata:
  name: kibana-7-6
  namespace: default
spec:
  version: 7.6.2
  count: 1
  elasticsearchRef:
    name: elasticsearch-7-6
    namespace: default
  http:
    tls:
      selfSignedCertificate:
        disabled: true
  podTemplate:
    spec:
      containers:
        - name: kibana
          resources:
            requests:
              memory: 0.2Gi
              cpu: 0.1
            limits:
              memory: 2Gi
              cpu: 1

---

apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: kibana-7-6
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
    - host: kibana-7-6.testing2.bbdops.com
      http:
        paths:
          - path: /
            backend:
              serviceName: kibana-7-6-kb-http
              servicePort: 5601
```



## 认证

ES 和 Kibana 的用户名都为：elastic，并且 ES 和 Kibana 的密码也都为：

```bash
kubectl get secret elasticsearch-6-8-es-elastic-user -o=jsonpath='{.data.elastic}' | base64 -d
```



## 坑

在配置好 ES 插件后，发现只有 test-kubenode-1 上可以安装插件并启动 ES，但是 其他机器上插件安装不上。

插件在所有宿主机上都可以下载，但是在除了 test-kubenode-1 之外主机上的容器里边就下载不了。。。。。。

最后将插件地址上传到公司服务器才搞定，插件地址：

http://mirrors.bbdops.com/list/public/elasticsearch/elasticsearch-analysis-ik-6.8.9.zip
http://mirrors.bbdops.com/list/public/elasticsearch/elasticsearch-analysis-ik-7.6.2.zip



