# 示例：使用 Redis 部署 PHP 留言板应用程序

文档地址：https://kubernetes.io/zh/docs/tutorials/stateless-application/guestbook/

redis-master-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-master
  labels:
    app: redis
spec:
  selector:
    matchLabels:
      app: redis
      role: master
      tier: backend
  replicas: 1
  template:
    metadata:
      labels:
        app: redis
        role: master
        tier: backend
    spec:
      containers:
        - name: master
          image: redis
          resources:
            requests:
              cpu: 100m
              memory: 100Mi
          ports:
            - containerPort: 6379
```

redis-master-service.yaml：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-master
  labels:
    app: redis
    role: master
    tier: backend
spec:
  ports:
    - port: 6379
      targetPort: 6379
  selector:
    app: redis
    role: master
    tier: backend
```

redis-slave-deployment.yaml:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-slave
  labels:
    app: redis
spec:
  selector:
    matchLabels:
      app: redis
      role: slave
      tier: backend
  replicas: 2
  template:
    metadata:
      labels:
        app: redis
        role: slave
        tier: backend
    spec:
      containers:
        - name: slave
          image: gcr.io/google_samples/gb-redisslave:v1
          resources:
            requests:
              cpu: 100m
              memory: 100Mi
          env:
            - name: GET_HOSTS_FROM
              value: dns
              # Using `GET_HOSTS_FROM=dns` requires your cluster to
              # provide a dns service. As of Kubernetes 1.3, DNS is a built-in
              # service launched automatically. However, if the cluster you are using
              # does not have a built-in DNS service, you can instead
              # access an environment variable to find the master
              # service's host. To do so, comment out the 'value: dns' line above, and
              # uncomment the line below:
              # value: env
          ports:
            - containerPort: 6379
```

redis-slave-service.yaml:

```
apiVersion: v1
kind: Service
metadata:
  name: redis-slave
  labels:
    app: redis
    role: slave
    tier: backend
spec:
  ports:
    - port: 6379
  selector:
    app: redis
    role: slave
    tier: backend
```

frontend-deployment.yaml:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  labels:
    app: guestbook
spec:
  selector:
    matchLabels:
      app: guestbook
      tier: frontend
  replicas: 3
  template:
    metadata:
      labels:
        app: guestbook
        tier: frontend
    spec:
      containers:
        - name: php-redis
          image: gcr.io/google-samples/gb-frontend:v4
          resources:
            requests:
              cpu: 100m
              memory: 100Mi
          env:
            - name: GET_HOSTS_FROM
              value: dns
              # Using `GET_HOSTS_FROM=dns` requires your cluster to
              # provide a dns service. As of Kubernetes 1.3, DNS is a built-in
              # service launched automatically. However, if the cluster you are using
              # does not have a built-in DNS service, you can instead
              # access an environment variable to find the master
              # service's host. To do so, comment out the 'value: dns' line above, and
              # uncomment the line below:
              # value: env
          ports:
            - containerPort: 80
```

frontend-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  type: NodePort
  ports:
    - port: 80
  selector:
    app: guestbook
    tier: frontend
```

提前准备好镜像，然后：

```bash
$ kubectl apply -f .
```

查看 pods：

```bash
$ kubectl get pods
```

查看 service：

```bash
$ kubectl get svc
```

添加 ingress，由于我之前是使用  `rke` 安装的 k8s 集群，所以自带了 ingress controller，即 nginx-ingress，所以这里可以直接使用 ingress，frontend-ingress.yaml：

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: frontend-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - http:
        paths:
          - path: /frontend
            backend:
              serviceName: frontend
              servicePort: 80
```

部署 ingress：

```bash
$ kubectl apply -f frontend-ingress.yaml
```

查看 ingress：

```bash
$ kubectl get ingress
NAME               HOSTS   ADDRESS                                                             PORTS   AGE
frontend-ingress   *       fueltank-1,fueltank-2,fueltank-3,fueltank-4,fueltank-5,fueltank-6   80      10m
```

然后浏览器打开：`https://fueltank-1/frontend` ，大功告成！！！

扩展 POD 数量：

```bash
 $ kubectl scale deployment frontend --replicas=5
```

缩小 POD 数量：

```bash
 $ kubectl scale deployment frontend --replicas=2
```

验证：

```bash
$ kubectl get pods
```

清理：

```bash
$ kubectl delete -f .
```

