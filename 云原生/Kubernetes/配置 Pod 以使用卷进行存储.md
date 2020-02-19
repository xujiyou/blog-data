# 配置 Pod 以使用卷进行存储

只要容器存在，容器的文件系统就会存在，因此当一个容器终止并重新启动，对该容器的文件系统改动将丢失。对于独立于容器的持久化存储，您可以使用[卷](https://kubernetes.io/docs/concepts/storage/volumes/)。这对于有状态应用程序尤为重要，例如键值存储（如 Redis）和数据库。

[`redis.yaml`](https://raw.githubusercontent.com/kubernetes/website/master/content/zh/examples/pods/storage/redis.yaml)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis
    volumeMounts:
    - name: redis-storage
      mountPath: /data/redis
  volumes:
  - name: redis-storage
    emptyDir: {}
```

```
$ kubectl apply -f https://k8s.io/examples/pods/storage/redis.yaml
$ kubectl get pod redis --watch
```

在另一终端：

```
$ kubectl exec -it redis -- /bin/bash
root@redis:/data# cd /data/redis/
root@redis:/data/redis# echo Hello > test-file

root@redis:/data/redis# apt-get update
root@redis:/data/redis# apt-get install procps
root@redis:/data/redis# ps aux

USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
redis        1  0.1  0.1  33308  3828 ?        Ssl  00:46   0:00 redis-server *:6379
root        12  0.0  0.0  20228  3020 ?        Ss   00:47   0:00 /bin/bash
root        15  0.0  0.0  17500  2072 ?        R+   00:48   0:00 ps aux
```

在 shell 终端中，结束 Redis 进程：

```shell
root@redis:/data/redis# kill <pid>
```

这时，留意上一个终端，POD 会重启，这是因为 Redis Pod 的 [restartPolicy](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.17/#podspec-v1-core) 为 `Always`。

检查刚才创建的文件还存在不：

```
$ kubectl exec -it redis -- /bin/bash
root@redis:/data/redis# cd /data/redis/
root@redis:/data/redis# ls
test-file
```

发现还存在

清理：

```
$ kubectl delete pod redis
```

