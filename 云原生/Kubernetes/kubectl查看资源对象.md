# kubectl查看资源对象

查看全部资源对象列表：

```bash
$ kubectl api-resources
```

查看全部版本：

```bash
$ kubectl api-versions
```

查看某一对象(比如 svc )的相信信息：

```bash
$ kubectl explain svc
```

通过以上我们能感觉到,以上好像并没有罗列出所有的api字段,实际上以上列出的仅是一级字段,一级字段可能还包含二级的,三级的字段,想要罗列出所有的字段,可以加上`--recursive`来列出所有可能的字段:

```bash
$ kubectl explain svc --recursive
```

通过上面`kubectl explain service --recursive`可以看到所有的api名称,但是以上仅仅是罗列了所有的api名称,如果想要知道某一个api名称的详细信息,则可以通过`kubectl explain <资源对象名称.api名称>`的方式来查看,比如以下示例可以查看到`service`下的`spec`下的`ports`字段的信息:

```
$ kubectl explain svc.spec.ports
```

## 并非所有对象都在命名空间中

大多数 kubernetes 资源（例如 Pod、Service、副本控制器等）都位于某些命名空间中。但是命名空间资源本身并不在命名空间中。而且底层资源，例如 [nodes](https://kubernetes.io/docs/admin/node) 和持久化卷不属于任何命名空间。

查看哪些 Kubernetes 资源在命名空间中，哪些不在命名空间中：

```shell
# In a namespace
kubectl api-resources --namespaced=true

# Not in a namespace
kubectl api-resources --namespaced=false
```