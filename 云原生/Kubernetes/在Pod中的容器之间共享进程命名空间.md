# 在 Pod 中的容器之间共享进程命名空间

文档地址：https://kubernetes.io/zh/docs/tasks/configure-pod-container/share-process-namespace/

此页面展示如何为 pod 配置进程命名空间共享。 当启用进程命名空间共享时，容器中的进程对该 pod 中的所有其他容器都是可见的。

您可以使用此功能来配置协作容器，比如日志处理 sidecar 容器，或者对那些不包含诸如 shell 等调试实用工具的镜像进行故障排查。

进程命名空间共享使用 `v1.PodSpec` 中的 `ShareProcessNamespace` 字段启用。例如：

[`share-process-namespace.yaml`](https://raw.githubusercontent.com/kubernetes/website/master/content/zh/examples/pods/share-process-namespace.yaml)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  shareProcessNamespace: true
  containers:
  - name: nginx
    image: nginx
  - name: shell
    image: busybox
    securityContext:
      capabilities:
        add:
        - SYS_PTRACE
    stdin: true
    tty: true
```

1. 在集群中创建 `nginx` pod：

   ```shell
   kubectl apply -f https://k8s.io/examples/pods/share-process-namespace.yaml
   ```

2. 获取容器 `shell`，执行 `ps`：

   ```shell
   kubectl attach -it nginx -c shell
   ```

   如果没有看到命令提示符，请按 enter 回车键。

   ```
   / # ps ax
   PID   USER     TIME  COMMAND
       1 root      0:00 /pause
       8 root      0:00 nginx: master process nginx -g daemon off;
      14 101       0:00 nginx: worker process
      15 root      0:00 sh
      21 root      0:00 ps ax
   ```

您可以在其他容器中对进程发出信号。例如，发送 `SIGHUP` 到 nginx 以重启工作进程。这需要 `SYS_PTRACE` 功能。

```
/ # kill -HUP 8
/ # ps ax
PID   USER     TIME  COMMAND
    1 root      0:00 /pause
    8 root      0:00 nginx: master process nginx -g daemon off;
   15 root      0:00 sh
   22 101       0:00 nginx: worker process
   23 root      0:00 ps ax
```

甚至可以使用 `/proc/$pid/root` 链接访问另一个容器镜像。

```
/ # head /proc/8/root/etc/nginx/nginx.conf

user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
```



Pod 共享许多资源，因此它们共享进程命名空间是很有意义的。 不过，有些容器镜像可能希望与其他容器隔离，因此了解这些差异很重要:

1. **容器进程不再具有 PID 1。** 在没有 PID 1 的情况下，一些容器镜像拒绝启动（例如，使用 `systemd` 的容器)，或者拒绝执行 `kill -HUP 1` 之类的命令来通知容器进程。在具有共享进程命名空间的 pod 中，`kill -HUP 1` 将通知 pod 沙箱（在上面的例子中是 `/pause`）。
2. **进程对 pod 中的其他容器可见。** 这包括 `/proc` 中可见的所有信息，例如作为参数或环境变量传递的密码。这些仅受常规 Unix 权限的保护。
3. **容器文件系统通过 `/proc/$pid/root` 链接对 pod 中的其他容器可见。** 这使调试更加容易，但也意味着文件系统安全性只受文件系统权限的保护。