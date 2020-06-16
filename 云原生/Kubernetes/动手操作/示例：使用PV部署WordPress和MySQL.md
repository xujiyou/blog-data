# 示例：使用 Persistent Volumes 部署 WordPress 和 MySQL

文档地址：https://kubernetes.io/zh/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/

PV 和 PVC 是一一对应的！！！

---

mysql-deployment.yaml:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress
    tier: mysql
  clusterIP: None

---

apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-volume
  labels:
    type: local
spec:
  storageClassName: mysql-storage-class-name
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/data/k8s/mysql"

---

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  labels:
    app: wordpress
spec:
  storageClassName: mysql-storage-class-name
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:5.6
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-pass
                  key: password
          ports:
            - containerPort: 3306
              name: mysql
          volumeMounts:
            - name: mysql-persistent-storage
              mountPath: /var/lib/mysql

      volumes:
        - name: mysql-persistent-storage
          persistentVolumeClaim:
            claimName: mysql-pv-claim
```

注意 PV 使用的目录要提前准备好，包括目录权限，集群中每个节点都要准备。

wordpress-deployment.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  ports:
    - port: 80
  selector:
    app: wordpress
    tier: frontend
  type: LoadBalancer

---

apiVersion: v1
kind: PersistentVolume
metadata:
  name: wordpress-volume
  labels:
    type: local
spec:
  storageClassName: wordpress-storage-class-name
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/data/k8s/wordpress"

---

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wp-pv-claim
  labels:
    app: wordpress
spec:
  storageClassName: wordpress-storage-class-name
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
        - name: wordpress
          image: wordpress:4.8-apache
          env:
            - name: WORDPRESS_DB_HOST
              value: wordpress-mysql
            - name: WORDPRESS_DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-pass
                  key: password
          ports:
            - containerPort: 80
              name: wordpress
          volumeMounts:
            - mountPath: /var/www/html
              name: wordpress-persistent-storage
      volumes:
        - name: wordpress-persistent-storage
          persistentVolumeClaim:
            claimName: wp-pv-claim
```

kustomization.yaml:

```yaml
secretGenerator:
  - name: mysql-pass
    literals:
      - password=xujiyou
resources:
  - wordpress-deployment.yaml
  - mysql-deployment.yaml
```

wordpress-ingress.yaml:

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: wordpress-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - http:
        paths:
          - path: /
            backend:
              serviceName: wordpress
              servicePort: 80
```

然后创建：

```
$ kubectl apply -k .
```

等一切创建好，然后浏览器打开：https://fueltank-1/ 就可以了。

查看 PV 中使用的目录，会发现里面有文件了！！！
