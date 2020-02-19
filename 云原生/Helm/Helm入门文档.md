# Helm 入门文档

Helm 就是 k8s 之上的 yum

安装过程略过。

使用 proxychains4 代理添加 Helm 库：

```
$ proxychains4 helm repo add stable https://kubernetes-charts.storage.googleapis.com/
```

查询：

```
$ helm search repo stable
NAME                                 	CHART VERSION	APP VERSION            	DESCRIPTION
stable/acs-engine-autoscaler         	2.2.2        	2.1.1                  	DEPRECATED Scales worker nodes within agent pools
stable/aerospike                     	0.3.2        	v4.5.0.5               	A Helm chart for Aerospike in Kubernetes
stable/airflow                       	5.2.4        	1.10.4                 	Airflow is a platform to programmatically autho...
stable/ambassador                    	5.3.0        	0.86.1                 	A Helm chart for Datawire Ambassador
stable/anchore-engine                	1.4.0        	0.6.0                  	Anchore container analysis and policy evaluatio...
# ... and many more
```

更新库：

```
$ helm repo update
```

查询 Elasticsearch

```
$ helm search repo stable/elasticsearch
NAME                         	CHART VERSION	APP VERSION	DESCRIPTION
stable/elasticsearch         	1.32.1       	6.8.2      	Flexible and powerful open source, distributed ...
stable/elasticsearch-curator 	2.1.3        	5.7.6      	A Helm chart for Elasticsearch Curator
stable/elasticsearch-exporter	2.1.1        	1.1.0      	Elasticsearch stats exporter for Prometheus
```

没有最新版，添加官方的 helm 库：

```
$ helm repo add elastic https://helm.elastic.co
```

安装：

```
$ helm install --name-template elasticsearch elastic/elasticsearch
```

没加命名空间，艹，删除之：

```
$ helm uninstall --name-template elasticsearch elastic/elasticsearch
```

指定命名空间和版本安装：

```
$ kubectl create namespace es-space
$ proxychains4 helm install --namespace es-space --name-template elasticsearch elastic/elasticsearch --set imageTag=7.5.1
```



## 命令自动补全

```bash
$ source <(helm completion bash)
$ echo "source <(helm completion bash)" >> ~/.bashrc
```

