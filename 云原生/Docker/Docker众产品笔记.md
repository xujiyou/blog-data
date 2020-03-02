# Docker 众产品笔记

官方文档：https://docs.docker.com/install/

Docker 的官方文档体验还是比较好的，白天模式和黑夜模式都比较好看。

## Docker Engine

不过多介绍了，很常用，是 Docker 的主要产品。

Docker 的架构是 C/S 的，服务端是一个名为 dockerd 的守护程序，命令 `docker` 是客户端。

他们之间通过 API 进行连接，最新的官方 API 参考：https://docs.docker.com/engine/api/v1.40/#

之前的 docker 叫 `docker` 或 `docker-engine` ，现在叫 docker-ce

CentOS 的安装介绍在：https://docs.docker.com/install/linux/docker-ce/centos/

## Docker Assemble

这是个试验性的特性

Docker Assemble（`docker assemble`）是一个插件，提供一种语言和框架感知工具，使用户能够将应用程序构建到优化的Docker容器中。使用Docker Assemble，用户可以通过从现有框架配置中自动检测所需信息来快速构建Docker映像，而无需提供配置信息（例如Dockerfile）。

目前Docker Assemble只支持以下应用程序框架：

- 使用[Maven](https://maven.apache.org/)构建系统时的[Spring Boot](https://spring.io/projects/spring-boot)
- [ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core)（带有C＃和F＃）