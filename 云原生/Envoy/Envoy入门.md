# Envoy 入门

为了深入学习 Istio，准备学习下 Envoy。Envoy 是 CNCF 中毕业的项目，具有很多高级特性。

Envoy：https://github.com/envoyproxy/envoy

参照教程：https://fuckcloudnative.io/posts/run-envoy-on-your-laptop/

## 安装

先安装 `docker-compose`

```bash
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.25.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

获取 envoy 的代码库：

```bash
$ git clone https://github.com/envoyproxy/envoy
$ cd envoy/examples/front-proxy
```

这个目录下有几个文件：

- service.py：定义了一个 Flask 后端服务
- service-envoy.yaml：定义了伴随 Flask 服务的 Envoy 服务
- Dockerfile-service：定义了在启动容器时同时启动 Flask 服务和其伴随的 Envoy 服务
- front-envoy.yaml：定义了前端代理的 Envoy 服务
- Dockerfile-frontenvoy：定义了前端代理的容器
- docker-compose.yaml：文件描述了如何构建、打包和运行前端代理与服务。

使用 docker-compose 启动容器：

```bash
$ docker-compose up --build -d
```

该命令将会启动一个前端代理和两个服务实例：service1 和 service2。

查看服务：

```bash
$ docker-compose ps
```















