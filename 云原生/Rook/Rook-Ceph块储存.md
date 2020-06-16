# Rook-Ceph 块储存

文档地址：https://rook.io/docs/rook/v1.2/ceph-block.html

官方文档提供了 CSI 和 Flex 两种实现。我搜了下相关资料，CSI 是未来，Flex 比较老。

两种方式目前都可以工作，推荐使用 CSI

---

## 创建 CephBlockPool 及 StorageClass

要使用 Ceph 的块储存，需要创建一个 `pool` ，在 Rook 中，这通过 `CephBlockPool` 来实现。

`StorageClass` 用于创建动态 PV，下面通过 CSI 的方式创建这俩资源。

storageclass.yaml：

```yaml
apiVersion: ceph.rook.io/v1
kind: CephBlockPool
metadata:
  name: replicapool
  namespace: rook-ceph
spec:
  failureDomain: host
  replicated:
    size: 3
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
   name: rook-ceph-block
# Change "rook-ceph" provisioner prefix to match the operator namespace if needed
provisioner: rook-ceph.rbd.csi.ceph.com
parameters:
    # clusterID is the namespace where the rook cluster is running
    clusterID: rook-ceph
    # Ceph pool into which the RBD image shall be created
    pool: replicapool

    # RBD image format. Defaults to "2".
    imageFormat: "2"

    # RBD image features. Available for imageFormat: "2". CSI RBD currently supports only `layering` feature.
    imageFeatures: layering

    # The secrets contain Ceph admin credentials.
    csi.storage.k8s.io/provisioner-secret-name: rook-csi-rbd-provisioner
    csi.storage.k8s.io/provisioner-secret-namespace: rook-ceph
    csi.storage.k8s.io/node-stage-secret-name: rook-csi-rbd-node
    csi.storage.k8s.io/node-stage-secret-namespace: rook-ceph

    # Specify the filesystem type of the volume. If not specified, csi-provisioner
    # will set default as `ext4`.
    csi.storage.k8s.io/fstype: xfs

# Delete the rbd volume when a PVC is deleted
reclaimPolicy: Delete
```

创建：

```bash
$ kubectl apply -f storageclass.yaml
```

通过使用上面的 StorageClass 来创建一个 PVC，创建 PVC 的过程中会自动创建 PV，这就是动态 PV。

相反，先创建 PV ，再利用  PV 创建 PVC 的方式叫静态 PV。

PV 和 PVC 是一一对应的。

pvc.yaml：

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pv-claim
  labels:
    type: rook-ceph-block-pvc
spec:
  storageClassName: rook-ceph-block
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

创建：

```bash
$ kubectl apply -f pvc.yaml
```



---



验证：

```bash
[admin@fueltank-1 ~]$ kubectl get CephBlockPool -n rook-ceph
NAME          AGE
replicapool   9m5s
[admin@fueltank-1 ~]$ kubectl get StorageClass
NAME               PROVISIONER                  RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
rook-ceph-block    rook-ceph.rbd.csi.ceph.com   Delete          Immediate           false                  9m17s
rook-ceph-bucket   ceph.rook.io/bucket          Delete          Immediate           false                  45h
[admin@fueltank-1 ~]$ kubectl get pvc
NAME          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
my-pv-claim   Bound    pvc-ab44b114-6c09-455e-bc32-36ecb24a4720   10Gi       RWO            rook-ceph-block   9m20s
[admin@fueltank-1 ~]$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS      REASON   AGE
pvc-ab44b114-6c09-455e-bc32-36ecb24a4720   10Gi       RWO            Delete           Bound    default/my-pv-claim   rook-ceph-block            9m15s
```

都创建成功了。
