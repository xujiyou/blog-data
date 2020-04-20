# Envoy 二进制构建及安装

Envoy 是 C++ 写的，构建系统是用的 `Bazel`，为了简化初始构建并快速入门，官方提供了基于 Ubuntu 16 的 Docker 镜像，这个镜像拥有构建和静态链接Envoy所需的一切。

构建 Envoy 需要：

- GCC 7+ or Clang/LLVM 7+ (for C++14 support).
- These [Bazel native](https://github.com/envoyproxy/envoy/blob/3a603ad3efe6d1124a73728bb5d7bfa51d0f8f52/bazel/repository_locations.bzl) dependencies.

CentOS 7 安装最新版本 GCC 9（需要特殊方法）：

```bash
$ sudo yum install centos-release-scl -y
$ sudo yum install devtoolset-9-gcc* -y
```

然后只能在新 bash 里面是用 gcc 9，退出这个 bash 还是用的系统的老 gcc：

```bash
$ scl enable devtoolset-9 bash
$ which gcc
$ gcc --version
```

CentOS 7 安装 Bazel：

```bash
$ wget https://copr.fedorainfracloud.org/coprs/vbatts/bazel/repo/epel-7/vbatts-bazel-epel-7.repo
$ sudo mv vbatts-bazel-epel-7.repo /etc/yum.repos.d
$ sudo yum install bazel -y
$ bazel version
```



## 官方提供的二进制文件

官方提供的二进制文件并没有放在官网里面，而是放在了：  https://www.getenvoy.io/

Yum 安装 Envoy 二进制文件：

```bash
$ sudo yum install -y yum-utils
$ sudo yum-config-manager --add-repo https://getenvoy.io/linux/centos/tetrate-getenvoy.repo
$ sudo yum install -y getenvoy-envoy
$ envoy --version
```



##getenvoy

获取 getenvoy 二进制文件。

```bash
$ curl -L https://getenvoy.io/cli | bash -s -- -b /usr/local/bin 
$ getenvoy --version
```

在当前命令行窗口启动一个 envoy：

```bash
$ curl -L https://getenvoy.io/samples/basic-front-proxy.yaml > basic-front-proxy.yaml
$ getenvoy run standard:1.14.1 -- --config-path ./basic-front-proxy.yaml
```

在另一个窗口访问：

```bash
$ curl -L -s -o /dev/null -vvv -H 'Host: bing.com' localhost:15001/
```

getenvoy 的方便之处是可以运行各个版本的 envoy！







