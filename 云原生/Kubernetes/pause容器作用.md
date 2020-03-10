# pause 容器作用

每一个 Pod 中，都会运行一个 pause 容器，但是在使用 kubectl 查看 Pod 中的容器时是看不到这个容器的，需要使用 `docker ps` 才能看到运行中的 pause 容器。

pause 是用 C 语言写成的，只有 200k ，非常小巧，也非常高效。

因为在 Kubernetes 中，是以 Pod 为基本单位的，一个 Pod 内可以包含多个 容器，但只靠 Docker 是不能让多个容器共享一个 Linux 命名空间的，所以这时候就需要用到 pause 容器了。

kubernetes中的pause容器主要为每个业务容器提供以下功能：

- 在pod中担任Linux命名空间共享的基础；
- 启用pid命名空间，开启init进程。

pause容器的PID是1

另外，pause 容器还有对接 CNI 的功能。