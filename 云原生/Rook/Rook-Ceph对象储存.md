# Rook Ceph对象储存

首先使用 Rook 在 k8s 集群中安装 Ceph。

---

## 安装 Ceph

官方文档：https://rook.io/docs/rook/v1.2/ceph-quickstart.html

```bash
$ git clone --single-branch --branch release-1.2 https://github.com/rook/rook.git
$ cd cluster/examples/kubernetes/ceph
$ kubectl create -f common.yaml
$ kubectl create -f operator.yaml
$ kubectl -n rook-ceph get pod
```

等待pod执行完成后，修改 cluster.yaml ，将地址修改为自己的地址：

```yaml
...
dataDirHostPath: /mnt/vde/rook
...
```

然后：

```bash
$ kubectl create -f cluster.yaml
```

等待集群创建完毕：

```bash
$ kubectl -n rook-ceph get pod
```





---



## 安装 Toolbox Pod

官方文档地址：https://rook.io/docs/rook/v1.2/ceph-toolbox.html

toolbox.yaml：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rook-ceph-tools
  namespace: rook-ceph
  labels:
    app: rook-ceph-tools
spec:
  replicas: 1
  selector:
    matchLabels:
      app: rook-ceph-tools
  template:
    metadata:
      labels:
        app: rook-ceph-tools
    spec:
      dnsPolicy: ClusterFirstWithHostNet
      containers:
      - name: rook-ceph-tools
        image: rook/ceph:v1.2.4
        command: ["/tini"]
        args: ["-g", "--", "/usr/local/bin/toolbox.sh"]
        imagePullPolicy: IfNotPresent
        env:
          - name: ROOK_ADMIN_SECRET
            valueFrom:
              secretKeyRef:
                name: rook-ceph-mon
                key: admin-secret
        securityContext:
          privileged: true
        volumeMounts:
          - mountPath: /dev
            name: dev
          - mountPath: /sys/bus
            name: sysbus
          - mountPath: /lib/modules
            name: libmodules
          - name: mon-endpoint-volume
            mountPath: /etc/rook
      # if hostNetwork: false, the "rbd map" command hangs, see https://github.com/rook/rook/issues/2021
      hostNetwork: true
      volumes:
        - name: dev
          hostPath:
            path: /dev
        - name: sysbus
          hostPath:
            path: /sys/bus
        - name: libmodules
          hostPath:
            path: /lib/modules
        - name: mon-endpoint-volume
          configMap:
            name: rook-ceph-mon-endpoints
            items:
            - key: data
              path: mon-endpoints
```

部署：

```bash
$ kubectl create -f toolbox.yaml
```

等待一段时间后，查看 Pod：

```bash
$ kubectl -n rook-ceph get pod -l "app=rook-ceph-tools"
```

待 POD 部署完成后，运行以下命令来进入 Toolbox Pod bash：

```bash
$ kubectl -n rook-ceph exec -it $(kubectl -n rook-ceph get pod -l "app=rook-ceph-tools" -o jsonpath='{.items[0].metadata.name}') bash
```

几个例子：

```bash
$ ceph status
$ ceph osd status
$ ceph df
$ rados df
```



---



## Ceph Dashboard

官方文档：https://rook.io/docs/rook/v1.2/ceph-dashboard.html

首先查看 Dashboard 服务地址：

```bash
[admin@fueltank-1 ~]$ kubectl get svc rook-ceph-mgr-dashboard -n rook-ceph
NAME                      TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
rook-ceph-mgr-dashboard   ClusterIP   10.43.203.169   <none>        8443/TCP   67m
```

为了从外部访问 Dashboard，这里选择使用 NodePort 的方式：

```bash
$ kubectl edit svc rook-ceph-mgr-dashboard -n rook-ceph
```

将 type 从 ClusterIP 改为 NodePort，修改完成后：

```bash
[admin@fueltank-1 ~]$ kubectl get svc rook-ceph-mgr-dashboard -n rook-ceph
NAME                      TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
rook-ceph-mgr-dashboard   NodePort   10.43.203.169   <none>        8443:31912/TCP   67m
```

浏览器访问 https://fueltank-1:31912

用户名为 admin，通过以下命令来生成密码：

```bash
$ kubectl -n rook-ceph get secret rook-ceph-dashboard-password -o jsonpath="{['data']['password']}" | base64 --decode && echo
```

我现在这里为 k3Vvq6diir

一会再为对象存储开启 Dashboard



---



## Ceph 对象储存

官方文档：https://rook.io/docs/rook/v1.2/ceph-object.html

首先创建对象存储

object.yaml：

```yaml
apiVersion: ceph.rook.io/v1
kind: CephObjectStore
metadata:
  name: my-store
  namespace: rook-ceph
spec:
  metadataPool:
    failureDomain: host
    replicated:
      size: 3
  dataPool:
    failureDomain: host
    erasureCoded:
      dataChunks: 2
      codingChunks: 1
  preservePoolsOnDelete: true
  gateway:
    type: s3
    sslCertificateRef:
    port: 80
    securePort:
    instances: 1
```

部署：

```bash
$ kubectl create -f object.yaml
## 等待 pod 创建好
$kubectl -n rook-ceph get pod -l app=rook-ceph-rgw
```

---

#### 创建 bucket

需要先创建 StorageClass

storageclass-bucket-delete.yaml：

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
   name: rook-ceph-bucket
provisioner: ceph.rook.io/bucket
reclaimPolicy: Delete
parameters:
  objectStoreName: my-store
  objectStoreNamespace: rook-ceph
  region: us-east-1
```

创建 StorageClass：

```bash
$ kubectl create -f storageclass-bucket-delete.yaml
```

再创建 Bucket Claim 

```yaml
apiVersion: objectbucket.io/v1alpha1
kind: ObjectBucketClaim
metadata:
  name: ceph-bucket
spec:
  generateBucketName: ceph-bkt
  storageClassName: rook-ceph-bucket
```

创建：

```bash
$ kubectl create -f object-bucket-claim-delete.yaml
```

创建完成后，下面来验证。

需要先报漏服务，按照上边的套路将 ClusterIP 改为 NodePort，完成后应该为：

```bash
[admin@fueltank-1 ceph]$ kubectl get svc rook-ceph-rgw-my-store -n rook-ceph
NAME                     TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
rook-ceph-rgw-my-store   NodePort   10.43.231.52   <none>        80:31904/TCP   70m
```

好了，服务地址就是 fueltank-1:31904

然后安装 s3cmd 

```bash
# CentOS
$ yum --assumeyes install s3cmd
# MacOS
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)" < /dev/null 2> /dev/null
$ brew install s3cmd
```

安装完成后进行测试，需要先设置环境变量：

```bash
$ export AWS_HOST=$(kubectl -n default get cm ceph-bucket -o yaml | grep BUCKET_HOST | awk '{print $2}')
$ export AWS_ACCESS_KEY_ID=$(kubectl -n default get secret ceph-bucket -o yaml | grep AWS_ACCESS_KEY_ID | awk '{print $2}' | base64 --decode)
$ export AWS_SECRET_ACCESS_KEY=$(kubectl -n default get secret ceph-bucket -o yaml | grep AWS_SECRET_ACCESS_KEY | awk '{print $2}' | base64 --decode)
```

获取 bucket 名字（这里官网的文档时错误的，bucket 名字需要手动获取）：

```bash
[admin@fueltank-1 ceph]$ kubectl describe ObjectBucketClaim ceph-bucket
Name:         ceph-bucket
Namespace:    default
Labels:       <none>
Annotations:  <none>
API Version:  objectbucket.io/v1alpha1
Kind:         ObjectBucketClaim
Metadata:
  Creation Timestamp:  2020-02-15T06:06:49Z
  Generation:          2
  Resource Version:    3531429
  Self Link:           /apis/objectbucket.io/v1alpha1/namespaces/default/objectbucketclaims/ceph-bucket
  UID:                 c496c0f3-f9f7-4e78-bc3e-5ab5eaf471f0
Spec:
  Object Bucket Name:    obc-default-ceph-bucket
  Additional Config:     <nil>
  Bucket Name:           ceph-bkt-c21d2dcb-2846-4016-b03e-d5ade037e532
  Canned Bucket Acl:     
  Generate Bucket Name:  ceph-bkt
  Ssl:                   false
  Storage Class Name:    rook-ceph-bucket
  Versioned:             false
Status:
  Phase:  bound
Events:   <none>
```

这里刚才创建的 bucket 名为 ceph-bkt-c21d2dcb-2846-4016-b03e-d5ade037e532

然后进行测试：

````bash
[admin@fueltank-1 ~]$ echo "Hello Rook" > /tmp/rookObj
[admin@fueltank-1 ~]$ s3cmd put /tmp/rookObj --no-ssl --host=fueltank-1:31904 --host-bucket=  s3://ceph-bkt-c21d2dcb-2846-4016-b03e-d5ade037e532
upload: '/tmp/rookObj' -> 's3://ceph-bkt-c21d2dcb-2846-4016-b03e-d5ade037e532/rookObj'  [1 of 1]
 11 of 11   100% in    0s    11.50 B/s  done
[admin@fueltank-1 ~]$ s3cmd get s3://ceph-bkt-c21d2dcb-2846-4016-b03e-d5ade037e532/rookObj /tmp/rookObj-download --no-ssl --host=fueltank-1:31904 --host-bucket= --force
download: 's3://ceph-bkt-c21d2dcb-2846-4016-b03e-d5ade037e532/rookObj' -> '/tmp/rookObj-download'  [1 of 1]
 11 of 11   100% in    0s   224.25 B/s  done
[admin@fueltank-1 ~]$ cat /tmp/rookObj-download
Hello Rook
````

完成。

这里需要注意的是，因为这里使用了三个环境变量（AWS_HOST，AWS_ACCESS_KEY_ID，AWS_SECRET_ACCESS_KEY），所以 s3cmd 命令不报错。

如果不想使用环境变量，可以使用配置文件，下列命令用于生成默认配置文件：

```bash
$ s3cmd --configure
```

其中，主要是配置 `access_key` 和 `secret_key` ，生成方式上边都有，就是那俩环境变量。



---



## 打开对象储存的 Dashboard

文档地址（在最后一节）：https://rook.io/docs/rook/v1.2/ceph-dashboard.html

（可选）创建用户，如果下面这个用户，上面创建 bucket 时还自动生成了一个用户，不过官方文档是自己创建了一个用户

object-user.yaml

```yaml
apiVersion: ceph.rook.io/v1
kind: CephObjectStoreUser
metadata:
  name: my-user
  namespace: rook-ceph
spec:
  store: my-store
  displayName: "my display name"
```

创建：

```bash
$ kubectl create -f object-user.yaml
```

创建完成后，打开对象储存的 Dashboard

```bash
# 进入 Toolbox bash
$ kubectl -n rook-ceph exec -it $(kubectl -n rook-ceph get pod -l "app=rook-ceph-tools" -o jsonpath='{.items[0].metadata.name}') bash

$ radosgw-admin user modify --uid=my-user --system
# 会展示出用户信息，其中就包括 access-key 和 secret-key
$ ceph dashboard set-rgw-api-user-id my-user
$ ceph dashboard set-rgw-api-access-key <access-key>
$ ceph dashboard set-rgw-api-secret-key <secret-key>
```

完成后，重启下 Dashboard（这步官网没有）：

```bash
$ ceph mgr module disable dashboard
$ ceph mgr module enable dashboard
```

重启完成后，刷新浏览器即可看到对象储存的 Dashboard
