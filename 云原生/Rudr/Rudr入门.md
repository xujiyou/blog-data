# Rudr 入门

参考：https://github.com/oam-dev/rudr/blob/master/docs/tutorials/deploy_and_update.md

安装

```bash
$ git clone https://github.com/oam-dev/rudr.git
$ cd rudr
$ helm install rudr charts/rudr
```

验证：

```bash
$ kubectl get pods
```

运行 Demo：

```bash
$ kubectl apply -f examples/helloworld-python-component.yaml
```

检查：

```bash
$ kubectl get componentschematics
$ kubectl get traits --all-namespaces
```



```bash
$ kubectl apply -f examples/first-app-config.yaml
```



## 总结

感觉就是把开发和运维分开。