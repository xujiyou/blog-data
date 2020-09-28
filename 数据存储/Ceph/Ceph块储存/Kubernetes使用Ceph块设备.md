# Kubernetes 使用 Ceph 块设备

教程：https://docs.ceph.com/docs/master/rbd/rbd-kubernetes/

创建一个 pool：

```bash
$ sudo ceph osd pool create kubernetes
```

查看 pool 中的 pg 数量、pgp数量、复制份数：

```bash
$ sudo ceph osd pool get kubernetes pg_num
$ sudo ceph osd pool get kubernetes pgp_num
$ sudo ceph osd dump | grep size | grep kubernetes
```

在 https://ceph.com/pgcalc/ 中计算 pool 的数量，然后使用以下语句修改：

```bash
$ sudo ceph osd pool set kubernetes pg_num 512
$ sudo ceph osd pool set kubernetes pgp_num 512
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
        key = AQCl1DRfftOWHBAA+VIQ7jVZO2Q0H4ERF6A1mg==
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
  namespace: ceph
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

注意，config.json 要是符合json格式，特别是 monitors 最后一项没有逗号！！！

创建：

```bash
$ kubectl create ns ceph
$ kubectl apply -f csi-config-map.yaml
```



## ceph-csi  Secret

创建一个 Secret，名为 `csi-rbd-secret.yaml`，内容如下：

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: csi-rbd-secret
  namespace: ceph
stringData:
  userID: kubernetes
  userKey: AQCl1DRfftOWHBAA+VIQ7jVZO2Q0H4ERF6A1mg==
```

创建：

```bash
$ kubectl apply -f csi-rbd-secret.yaml
```



## 配置 ceph-csi 插件

ceph-csi 官方代码库：https://github.com/ceph/ceph-csi

首先创建 ceph-csi 插件使用到的 ServiceAccount 和 RBAC 信息（注意修改文件中的命名空间）：

```bash
$ kubectl apply -f https://raw.githubusercontent.com/ceph/ceph-csi/master/deploy/rbd/kubernetes/csi-provisioner-rbac.yaml -n ceph
$ kubectl apply -f https://raw.githubusercontent.com/ceph/ceph-csi/master/deploy/rbd/kubernetes/csi-nodeplugin-rbac.yaml -n ceph
```

最后，创建 ceph-csi provisioner 和 node 插件：

```bash
$ wget https://raw.githubusercontent.com/ceph/ceph-csi/master/deploy/rbd/kubernetes/csi-rbdplugin-provisioner.yaml 
$ kubectl apply -f csi-rbdplugin-provisioner.yaml -n ceph
$ wget https://raw.githubusercontent.com/ceph/ceph-csi/master/deploy/rbd/kubernetes/csi-rbdplugin.yaml
$ kubectl apply -f csi-rbdplugin.yaml -n ceph
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
  namespace: ceph
```

创建：

```bash
$ kubectl apply -f kms-config.yaml
```

等待 Pod 创建完成：

```bash
$ kubectl get pods -n ceph --watch
```

我这里 pod 全部创建完成！



## 测试使用

创建一个 `StorageClass`，文件名是 ceph-storage-class.yaml

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
   name: ceph-storage-class
provisioner: rbd.csi.ceph.com
parameters:
   clusterID: 150147a4-72f8-456d-8b8c-5204fb866be4
   pool: kubernetes
   fsType: xfs
   csi.storage.k8s.io/provisioner-secret-name: csi-rbd-secret
   csi.storage.k8s.io/provisioner-secret-namespace: ceph
   csi.storage.k8s.io/node-stage-secret-name: csi-rbd-secret
   csi.storage.k8s.io/node-stage-secret-namespace: ceph
reclaimPolicy: Retain
allowVolumeExpansion: true
mountOptions:
   - discard
```

这里的 `clusterID` 可以通过 `ceph status` 获取。`csi-rbd-secret` 就是上面创建的 secret。

创建：

```bash
$ kubectl apply -f ceph-storage-class.yaml
```



创建一个 PVC，命名为 `test-pvc.yaml`

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
  storageClassName: ceph-storage-class
```

创建：

```bash
$ kubectl apply -f test-pvc.yaml
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



## 错误

创建 PVC 时没问题，但是在使用时会有一个错误，详情见  [Ceph块设备挂载.md](Ceph块设备挂载.md) 



## 查看块设备内容

有个地方不理解，PV 的回收策略是 DELETE，但是当块 PVC 被删除时，PV 并不会自动删除，而是变成了 Released 状态，再等一段时间，看看会不会自动删除。

PV 不会自动删除，同理，块设备也不会被删除，有块设备在，就可以查看块设备里边的内容！！！

查看块设备：

```bash
$ sudo rbd ls kubernetes -l
```

加一个 `-l`参数可以看出块设备有没有被使用，方便删除没有使用的块设备！

映射块设备为本地磁盘：

```bash
$ sudo rbd device map kubernetes/csi-vol-49b1dcdf-a6f8-11ea-bf90-0a580a2a021d
```

查看映射到本地的设备列表：

```bash
$ sudo rbd device list
$ sudo fdisk -l
```

挂载到文件系统：

```bash
$ sudo mkdir /mnt/rbd1
$ sudo mount /dev/rbd1 /mnt/rbd1
```

这样就可以查看块设备里面的内容了：

```bash
$ ls /mnt/rbd1
```

查看块设备的文件系统：

```bash
$ mount | grep /dev/rbd1
$ df -hT
```

取消挂载：

```bash
$ sudo umount /dev/rbd1
```

查看所有磁盘的挂载结果：

```bash
$ sudo lsblk
```

取消映射：

```bash
$ sudo rbd device unmap kubernetes/csi-vol-49b1dcdf-a6f8-11ea-bf90-0a580a2a021d
```

取消挂载到本地文件系统，并取消映射后，Kubernetes 就可以继续使用这个块设备了。









