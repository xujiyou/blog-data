# Docker API (三) - Images

文档地址在：https://docs.docker.com/engine/api/v1.40/#tag/Image

#### 获取镜像列表

```http
GET /images/json
```

相当于 `docker image ls`



#### 构建镜像

```http
POST /build
```

相当于 `docker image build`



#### 删除构建缓存

```http
POST /build/prune
```



#### 创建镜像

```http
POST /images/create
```

相当于 `docker image pull`



#### 查看镜像信息

```http
GET /images/{name}/json
```

相当于 `docker image inspect`



#### 获取镜像的历史信息

```
GET /images/{name}/history
```

相当于 `docker image history`



#### Push an image

```http
POST /images/{name}/push
```

相当于 `docker image push`



#### Tag an image

给镜像打标签

```http
POST /images/{name}/tag
```

相当于 `docker image tag`



#### 删除镜像

```http
DELETE /images/{name}
```

相当于 `docker rmi` 和 `docker image rm`



#### 搜索镜像

```http
GET /images/search
```

相当于 `docker search`



#### 删除未使用的镜像

```http
POST /images/prune
```

相当于 `docker image prune`



#### 从运行中的容器中创建镜像

```http
POST /commit
```

相当于 `docker commit`



#### 导出一个镜像

```http
GET /images/{name}/get
```

例子：

```bash
$ curl  http://127.0.0.1:2376/images/calico/node:latest/get -o calico.tar
```

相当于 `docker image save`



#### 导出一堆镜像

```http
GET /images/get
```



#### 加载镜像 - 导入镜像

```http
POST /images/load
```

相当于 `docker image load`



#### 获取远程库中的镜像信息

```http
GET /distribution/{name}/json
```











