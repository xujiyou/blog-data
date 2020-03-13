# 使用 ConfigMap 来配置 Redis

教程地址：https://kubernetes.io/zh/docs/tutorials/configuration/

下面是简化版

redis-config:

```
maxmemory 2mb
maxmemory-policy allkeys-lru
```

redis-pod.yaml:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
    - name: redis
      image: redis
      env:
        - name: MASTER
          value: "true"
      ports:
        - containerPort: 6379
      resources:
        limits:
          cpu: "0.1"
      volumeMounts:
        - mountPath: /redis-master-data
          name: data
        - mountPath: /redis-master
          name: config
  volumes:
    - name: data
      emptyDir: {}
    - name: config
      configMap:
        name: example-redis-config
        items:
          - key: redis-config
            path: redis.conf

```

kustomization.yaml:

```yaml
configMapGenerator:
  - name: example-redis-config
    files:
      - redis-config
resources:
  - redis-pod.yaml
```



创建：

```bash
$ kubectl apply -k .
```

检查：

```bash
$ kubectl get -k .
```

检查是否挂载正确：

```bash
[admin@fueltank-1 config_map_example]$ kubectl exec -it redis bash
root@redis:/data# ls
root@redis:/data# ls /
bin  boot  data  dev  etc  home  lib  lib64  media  mnt  opt  proc  redis-master  redis-master-data  root  run	sbin  srv  sys	tmp  usr  var
root@redis:/data# ls /redis-master
redis.conf
root@redis:/data# ls /redis-master/redis.conf
/redis-master/redis.conf
root@redis:/data# cat /redis-master/redis.conf
maxmemory 2mb
maxmemory-policy allkeys-lru
```

删除创建的资源：

```bash
$ kubectl delete -k .
```

