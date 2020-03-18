# CRI Tools

CRI 的工具，可以查看项目：https://github.com/kubernetes-sigs/cri-tools

主要包含两个工具 crictl 和 critest。可以在 releases 中下载。

## crictl

文档：https://github.com/kubernetes-sigs/cri-tools/blob/master/docs/crictl.md

先查看帮助信息：

```
$ crictl -h
NAME:
   crictl - client for CRI

USAGE:
   crictl [global options] command [command options] [arguments...]

VERSION:
   v1.17.0

COMMANDS:
   attach              Attach to a running container
   create              Create a new container
   exec                Run a command in a running container
   version             Display runtime version information
   images, image, img  List images
   inspect             Display the status of one or more containers
   inspecti            Return the status of one or more images
   imagefsinfo         Return image filesystem info
   inspectp            Display the status of one or more pods
   logs                Fetch the logs of a container
   port-forward        Forward local port to a pod
   ps                  List containers
   pull                Pull an image from a registry
   run                 Run a new container inside a sandbox
   runp                Run a new pod
   rm                  Remove one or more containers
   rmi                 Remove one or more images
   rmp                 Remove one or more pods
   pods                List pods
   start               Start one or more created containers
   info                Display information of the container runtime
   stop                Stop one or more running containers
   stopp               Stop one or more running pods
   update              Update one or more running containers
   config              Get and set crictl options
   stats               List container(s) resource usage statistics
   completion          Output shell completion code
   help, h             Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --config value, -c value            Location of the client config file. If not specified and the default does not exist, the program's directory is searched as well (default: "/etc/
crictl.yaml") [$CRI_CONFIG_FILE]
   --debug, -D                         Enable debug mode
   --image-endpoint value, -i value    Endpoint of CRI image manager service [$IMAGE_SERVICE_ENDPOINT]
   --runtime-endpoint value, -r value  Endpoint of CRI container runtime service (default: "unix:///var/run/dockershim.sock") [$CONTAINER_RUNTIME_ENDPOINT]
   --timeout value, -t value           Timeout of connecting to the server (default: 2s)
   --help, -h                          show help
   --version, -v                       print the version
```

先来看下边的几个选项：



#### --version, -v

打印 crictl 自己的版本：

```bash
$ crictl --version
crictl version v1.17.0
```

#### --help, -h

查看帮助

#### --timeout value, -t value

连接 server 的超时时间 默认2s

#### --runtime-endpoint value, -r value

运行时服务的地址，默认 unix:///var/run/dockershim.sock

##### --image-endpoint value, -i value

镜像管理服务的地址

#### --config value, -c value

指定配置文件。 如果未指定且默认值不存在，则还将搜索程序的目录。默认 /etc/crictl.yaml



其他的子命令可以当作一个 docker 命令来看待，很简单，很好理解。



## critest

critest -h

#### 

























