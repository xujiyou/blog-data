# Kubernetes API 学习 (十一)- authentication

查看资源对象列表：

```
$ kubectl api-resources --api-group=authentication.k8s.io
NAME           SHORTNAMES   APIGROUP                NAMESPACED   KIND
tokenreviews                authentication.k8s.io   false        TokenReview
```

只有一个 TokenReview 对象，源码在 API 源码中的 authentication 包中。

这个东西只是用在 apiserver 的 webhook 上，网上资料不多。不学了。