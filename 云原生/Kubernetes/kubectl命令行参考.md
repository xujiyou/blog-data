# kubectl 命令行参考

官方文档地址：https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands

port-forward 可以直接映射 Pod 的端口。

```bash
$ kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=jaeger -o jsonpath='{.items[0].metadata.name}') 15032:16686 &
```

