# Docker API (五) - Volume



#### 列出所有的卷

```html
GET /volumes
```

相当于 `docker volume ls`



#### 创建一个卷

```http
POST /volumes/create
```

相当于 `docker volume create`



#### 查看一个卷的详细信息

```http
GET /volumes/{name}
```

相当于 `docker volume inspect`



#### 删除一个卷

```http
DELETE /volumes/{name}
```

相当于 `docker volume rm`



#### 删除所有没在使用的卷

```http
POST /volumes/prune	
```

相当于 `docker volume prune`



