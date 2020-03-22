# Docker API (四) - Network



#### 获取网络列表

```http
GET /networks
```

相当于 `docker network ls`



#### 获取某个网络的相信信息

```html
GET /networks/{id}
```

相当于 `docker network inspect`



#### 删除一个网络

```http
DELETE /networks/{id}
```

相当于 `docker network rm`



#### 创建一个网络

```http
POST /networks/create
```

相当于 `docker network create`



#### 将容器连接到网络上

```http
POST /networks/{id}/connect
```

相当于 `docker network connect`



#### 将容器和网络断开连接

```http
POST /networks/{id}/disconnect
```

相当于 `docker network disconnect`



#### 删除没有使用的网络

```
POST /networks/prune
```

相当于 `docker network prune`



