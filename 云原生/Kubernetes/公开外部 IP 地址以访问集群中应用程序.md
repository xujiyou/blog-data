# 公开外部 IP 地址以访问集群中应用程序

文档地址：https://kubernetes.io/zh/docs/tutorials/stateless-application/expose-external-ip-address/

由于 `gcr.io/google-samples/node-hello:1.0` 被墙，需要另想他法：

```
$ docker pull gcr.azk8s.cn/google-samples/node-hello:1.0
$ docker tag gcr.azk8s.cn/google-samples/node-hello:1.0 gcr.io/google-samples/node-hello:1.0
```

命令行创建 Deployment：

```
$ kubectl run hello-world --replicas=5 --labels="run=load-balancer-example" --image=gcr.io/google-samples/node-hello:1.0  --port=8080
```

显示有关 Deployment 的信息：

```bash
$ kubectl get deployments hello-world
$ kubectl describe deployments hello-world
```

显示有关 ReplicaSet 对象的信息：

```bash
$ kubectl get replicasets
$ kubectl describe replicasets
```

创建公开 deployment 的 Service 对象：

```bash
$ kubectl expose deployment hello-world --type=LoadBalancer --name=my-service
```

显示有关 Service 的信息：

```bash
$ kubectl get services my-service
```

显示有关 Service 的详细信息：

```bash
$ kubectl describe services my-service
Name:                     my-service
Namespace:                default
Labels:                   run=load-balancer-example
Annotations:              <none>
Selector:                 run=load-balancer-example
Type:                     LoadBalancer
IP:                       10.43.15.96
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  32395/TCP
Endpoints:                10.42.0.12:8080,10.42.2.18:8080,10.42.3.53:8080 + 2 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

调用：

```
$ curl http://10.42.0.12:8080
Hello Kubernetes!
```

