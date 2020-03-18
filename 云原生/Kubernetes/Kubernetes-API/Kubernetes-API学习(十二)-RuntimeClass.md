# Kubernetes API 学习 (十二) - RuntimeClass

RuntimeClass 是跟 CRI 关联的。

可以查看公开课：https://edu.aliyun.com/lesson_1651_13101?spm=5176.254948.1387840.31.2c0acad2WRrabG#_13101

kubectl查看：

```bash
$ kubectl api-resources --api-group=node.k8s.io
NAME             SHORTNAMES   APIGROUP      NAMESPACED   KIND
runtimeclasses                node.k8s.io   false        RuntimeClass
```

