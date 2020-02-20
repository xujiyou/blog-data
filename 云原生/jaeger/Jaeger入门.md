# Jaeger 入门

安装 Istio 后，执行：

```
$ kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=jaeger -o jsonpath='{.items[0].metadata.name}') 15032:16686 &
```

然后就可以通过 `http://localhost:15032`来访问jaeger的UI