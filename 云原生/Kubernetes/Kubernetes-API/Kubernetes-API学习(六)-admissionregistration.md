# Kubernetes - API 学习（六）- admissionregistration

查看对象：

```bash
$ kubectl api-resources --api-group=admissionregistration.k8s.io
NAME                              SHORTNAMES   APIGROUP                       NAMESPACED   KIND
mutatingwebhookconfigurations                  admissionregistration.k8s.io   false        MutatingWebhookConfiguration
validatingwebhookconfigurations                admissionregistration.k8s.io   false        ValidatingWebhookConfiguration
```

两个 Webhook 对象，暂时先不看了。