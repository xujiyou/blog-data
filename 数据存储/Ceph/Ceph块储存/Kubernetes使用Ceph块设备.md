# Kubernetes 使用 Ceph 块设备

教程：https://docs.ceph.com/docs/master/rbd/rbd-kubernetes/

创建一个 pool：

```bash
$ sudo ceph osd pool create kubernetes
```

对 pool 进行初始化：

```bash
$ sudo rbd pool init kubernetes
```

创建一个 auth：

```bash
$ sudo ceph auth get-or-create client.kubernetes mon 'profile rbd' osd 'profile rbd pool=kubernetes' mgr 'profile rbd pool=kubernetes'
```

返回：

```
[client.kubernetes]
        key = AQC9OZ1ezHSVOhAA+JLwECCL6RkNZLd3uKnLqw==
```



## ceph-csi ConfiMap

ceph-csi 需要一个存储在 Kubernetes 中的 ConfigMap 对象来定义 Ceph 集群的 monitor 地址：

```bash
$ sudo ceph mon dump
```

响应如下：

```
dumped monmap epoch 1
epoch 1
fsid 418765c6-2776-4d19-8ab3-1d887278256e
last_changed 2020-04-14T11:25:56.933858+0800
created 2020-04-14T11:25:56.933858+0800
min_mon_release 15 (octopus)
0: [v2:172.20.20.145:3300/0,v1:172.20.20.145:6789/0] mon.fueltank-3
1: [v2:172.20.20.162:3300/0,v1:172.20.20.162:6789/0] mon.fueltank-1
2: [v2:172.20.20.179:3300/0,v1:172.20.20.179:6789/0] mon.fueltank-2
```

创建一个 ConfigMap，名为 `csi-config-map.yaml`，内容如下：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: ceph-csi-config
data:
  config.json: |-
    [
      {
        "clusterID": "418765c6-2776-4d19-8ab3-1d887278256e",
        "monitors": [
          "172.20.20.162:6789",
          "172.20.20.179:6789",
          "172.20.20.145:6789"
        ]
      }
    ]
```

其中的 monitors 的地址是我本地的地址，clusterID 是通过 `ceph status` 获取的！

创建：

```bash
$ kubectl apply -f csi-config-map.yaml
```



## ceph-csi  Secret

创建一个 Secret，名为 `csi-rbd-secret.yaml`，内容如下：

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: csi-rbd-secret
  namespace: default
stringData:
  userID: kubernetes
  userKey: AQC9OZ1ezHSVOhAA+JLwECCL6RkNZLd3uKnLqw==
```

创建：

```bash
$ kubectl apply -f csi-rbd-secret.yaml
```



## 配置 ceph-csi 插件

首先创建 ceph-csi 插件使用到的 ServiceAccount 和 RBAC 信息：

```bash
$ kubectl apply -f https://raw.githubusercontent.com/ceph/ceph-csi/master/deploy/rbd/kubernetes/csi-provisioner-rbac.yaml
$ kubectl apply -f https://raw.githubusercontent.com/ceph/ceph-csi/master/deploy/rbd/kubernetes/csi-nodeplugin-rbac.yaml
```

最后，创建 ceph-csi provisioner 和 node 插件：

```bash
$ wget https://raw.githubusercontent.com/ceph/ceph-csi/master/deploy/rbd/kubernetes/csi-rbdplugin-provisioner.yaml
$ kubectl apply -f csi-rbdplugin-provisioner.yaml
$ wget https://raw.githubusercontent.com/ceph/ceph-csi/master/deploy/rbd/kubernetes/csi-rbdplugin.yaml
$ kubectl apply -f csi-rbdplugin.yaml
```

#### 错误

错误描述：https://github.com/ceph/ceph-csi/issues/834

解决方法：创建 kms-config.yaml，内容如下：

```yacas
apiVersion: v1
kind: ConfigMap
data:
  config.json: |-
    {
      "vault-test": {
        "encryptionKMSType": "vault",
        "vaultAddress": "http://vault.default.svc.cluster.local:8200",
        "vaultAuthPath": "/v1/auth/kubernetes/login",
        "vaultRole": "csi-kubernetes",
        "vaultPassphraseRoot": "/v1/secret",
        "vaultPassphrasePath": "ceph-csi/",
        "vaultCAVerify": "false"
      }
    }
metadata:
  name: ceph-csi-encryption-kms-config
```

创建：

```bash
$ kubectl apply -f kms-config.yaml
```

等待 Pod 创建完成：

```bash
$ kubectl get pods --watch
```

我这里 pod 全部创建完成！



## 测试使用

创建一个 `StorageClass`，文件名是 `csi-rbd-sc.yaml`

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
   name: csi-rbd-sc
provisioner: rbd.csi.ceph.com
parameters:
   clusterID: b9127830-b0cc-4e34-aa47-9d1a2e9949a8
   pool: kubernetes
   csi.storage.k8s.io/provisioner-secret-name: csi-rbd-secret
   csi.storage.k8s.io/provisioner-secret-namespace: default
   csi.storage.k8s.io/node-stage-secret-name: csi-rbd-secret
   csi.storage.k8s.io/node-stage-secret-namespace: default
reclaimPolicy: Delete
mountOptions:
   - discard
```

这里的 `clusterID` 可以通过 `ceph status` 获取。`csi-rbd-secret` 就是上面创建的 secret。

创建：

```bash
$ kubectl apply -f csi-rbd-sc.yaml 
```



创建一个 PVC，命名为 `raw-block-pvc.yaml`

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: raw-block-pvc
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Block
  resources:
    requests:
      storage: 1Gi
  storageClassName: csi-rbd-sc
```

创建：

```bash
$ kubectl apply -f raw-block-pvc.yaml
```

查看：

```bash
$ kubectl get sc
$ kubectl get pv
$ kubectl get pvc
```

查看块设备，发现已经创建了一个了：

```bash
$ sudo rbd ls kubernetes
```





