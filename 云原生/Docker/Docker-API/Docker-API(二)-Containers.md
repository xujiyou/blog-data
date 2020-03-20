# Docker API (二) - Containers

https://github.com/docker/docker-ce/blob/master/components/engine/api/types/types.go 这里有很多请求响应的结构体。

下面的 REST API 对应 `docker container --help` 下的命令。

#### 获取容器列表

```http
GET /containers/json
```

对应 `docker container ps`



#### 创建容器

```http
POST /containers/create
```

这个创建容器的 API 的请求参数很有意思，和 Dockerfile 有关，具体查看：https://docs.docker.com/engine/api/v1.40/#operation/ContainerCreate

对应 `docker container build`



#### Inspect a container，获取单个容器信息：

```http
GET /containers/{id}/json
```

对应 `docker container inspect`



#### List processes running inside a container，获取容器内部的进程信息：

```http
GET /containers/{id}/top
```

这条不支持 windows，相当于 `docker container top`，比如：

```bash
$ curl  http://127.0.0.1:2376/containers/cbef28a24c06/top | json_reformat 
$ docker top cbef28a24c06
```





#### 获取容器日志

```http
GET /containers/{id}/logs
```

相当于 `docker container logs`



#### Get changes on a container’s filesystem，获取容器内文件系统的修改信息

```http
GET /containers/{id}/changes
```

这个API有意思，可以测试一下：

```bash
$ curl  http://127.0.0.1:2376/containers/cbef28a24c06/changes | json_reformat
[
    {
        "Path": "/etc",
        "Kind": 0
    },
    {
        "Path": "/etc/coredns",
        "Kind": 1
    },
    {
        "Path": "/var",
        "Kind": 1
    },
    {
        "Path": "/var/run",
        "Kind": 1
    },
    {
        "Path": "/var/run/secrets",
        "Kind": 1
    },
    {
        "Path": "/var/run/secrets/kubernetes.io",
        "Kind": 1
    },
    {
        "Path": "/var/run/secrets/kubernetes.io/serviceaccount",
        "Kind": 1
    }
]
```

Kind 的含义是：

- `0`: Modified
- `1`: Added
- `2`: Deleted

相当于 `docker container diff` :

```bash
$ docker diff cbef28a24c06
C /etc
A /etc/coredns
A /var
A /var/run
A /var/run/secrets
A /var/run/secrets/kubernetes.io
A /var/run/secrets/kubernetes.io/serviceaccount
```



####打包容器文件系统

```http
GET /containers/{id}/export
```

实例：

```bash
$ curl  http://127.0.0.1:2376/containers/cbef28a24c06/export -o one.tar
```

相当于 `docker container export` :

```bash
$ docker export cbef28a24c06 -o two.tar
```



#### Get container stats based on resource usage

查看容器内进程的状态

```
GET /containers/{id}/stats
```

相当于 `docker container stats`:

```bash
$ docker stats cbef28a24c06
$ curl http://127.0.0.1:2376/containers/cbef28a24c06/stats
```



#### Resize a container TTY

```http
POST /containers/{id}/resize
```

好像跟 `docker container exec` 有些关系。



#### Start a container

启动一个或多个停止的容器。

```http
POST /containers/{id}/start
```

相当于 `docker container start`



#### Stop a container

停止一个或多个容器

```http
POST /containers/{id}/stop
```

相当于 `docker container stop`



#### Restart a container

重启一个或多个容器

```http
POST /containers/{id}/restart
```

相当于 `docker container restart`



#### Kill a container

干掉一个容器

```http
POST /containers/{id}/kill
```

相当于 `docker container kill`



#### Update a container

更新容器的配置，而不需要重建。

```
POST /containers/{id}/update
```

相当于 `docker container update`



#### Rename a container

重命名一个容器

```http
POST /containers/{id}/rename
```

相当于 `docker container rename`



#### Pause a container

挂起容器

```http
POST /containers/{id}/pause
```

相当于 `docker container pause`



#### Unpause a container

恢复一个被挂起的容器

```http
POST /containers/{id}/unpause
```

相当于 `docker container unpause`



#### Attach to a container

```http
POST /containers/{id}/attach
```

相当于 `docker container attach` 。

个人感觉 attach 比较鸡肋，没 exec 好用。



#### Attach to a container via a websocket

用 websocket 实现双向连接

```http
GET /containers/{id}/attach/ws
```



#### Wait for a container

阻塞直到容器退出并返回退出码

```http
POST /containers/{id}/wait
```

相当于 `docker container wait`



#### 删除容器

```http
DELETE /containers/{id}
```

相当于 `docker container rm`



#### Get information about files in a container

```
HEAD /containers/{id}/archive
```



#### 获取容器内的文件信息

```http
GET /containers/{id}/archive
```

例子：

```
curl  http://127.0.0.1:2376/containers/cbef28a24c06/archive?path=/var/run/secrets
```

相当于 `docker container cp` 命令。



#### 将本地文件拷贝至容器

```http
PUT /containers/{id}/archive
```

相当于 `docker container cp` 命令。



#### 删除已经停止的容器

```http
POST /containers/prune
```

相当于 `docker container prune` 命令。













