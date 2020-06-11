# 在Kubernetes上运行Spark

先构建镜像：

```
./bin/docker-image-tool.sh -r registry.prod.bbdops.com/common -t 0.0.1 build
```

要构建很长时间。生成了三个镜像，分别是 spark、spark-py、spark-r

上传镜像：

```
./bin/docker-image-tool.sh -r registry.prod.bbdops.com/common -t 0.0.1 push
```



## 测试运行

创建命名空间：

```bash
$ kubectl create ns spark
```

创建 ServiceAccount：

```bash
$ kubectl create serviceaccount spark -n spark
```

创建 ClusterRoleBinding，赋予 ServiceAccount 访问 kube-apiserver 某些资源的权限：

```bash
$ kubectl create clusterrolebinding spark-role --clusterrole=edit --serviceaccount=spark:spark --namespace=spark
```

执行 Demo：

```bash
$ ./bin/spark-submit \
 --master k8s://drift-1:6443 \
 --deploy-mode cluster \
 --name spark-pi \
 --class org.apache.spark.examples.SparkPi \
 --conf spark.executor.instances=5 \
 --conf spark.kubernetes.container.image=registry.prod.bbdops.com/common/spark:0.0.1 \
 --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark \
 --conf spark.kubernetes.namespace=spark \
 local:///opt/spark/examples/jars/spark-examples_2.11-2.4.6.jar
```

这里需要注意 spark-examples_2.11-2.4.6.jar 这个 jar 包是在 registry.prod.bbdops.com/common/spark:0.0.1 镜像内部的，如果不想每次都重新弄镜像，可以考虑把 jar 包放到 HDFS，然后从 HDFS 读取 jar 包。



查看任务执行过程：

```bash
$ kubectl get pods -n spark --watch
```

查看执行日志：

```bash
$ kubectl logs spark-pi-1591877485022-driver -n spark
```

执行完成后，pod 的状态会变成 Completed，在 spark-submit 窗口中会显示 Succeeded。 