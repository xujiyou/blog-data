# Kubernetes API 学习（一）- 介绍

我想根据官方的 API 源码，API 官方文档，kubectl 工具，curl 等工具来从头到尾的学习一下 Kubernetes API。

API 源码地址：https://github.com/kubernetes/api

API 官方文档：https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.17/

kubectl 获取集群所有的资源对象：

```bash
$ kubectl api-resources
```

kubectl 获取所有k8s的资源对象的版本：

```bash
$ kubectl api-versions | grep k8s
```

关于 API 分类，可以直接看源码中的目录分类，如下：

![image-20200312201423139](../../../resource/image-20200312201423139.png)

 