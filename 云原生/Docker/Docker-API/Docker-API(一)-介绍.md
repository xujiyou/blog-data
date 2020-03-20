# Docker API(一) - 介绍

最新 API 地址：https://docs.docker.com/engine/api/v1.40/

首先要打开 docker 的 tcp 端口，才能访问 docker 的 REST API，打开方式请看： [Docker远程连接.md](Docker远程连接.md) 

CentOS 安装 json 工具：

```bash
$ sudo yum install -y yajl
```

MacOS 安装 json 工具：

```bash
$ brew install yajl
```



测试连接：

```bash
 $ curl  http://127.0.0.1:2376/version
```



Docker API 的官方文档也是使用 swagger 生成的，swagger.yaml 在：https://github.com/docker/docker-ce/blob/master/components/engine/api/swagger.yaml

