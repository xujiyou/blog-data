# Docker API (七) - system



#### 检查认证

```http
POST /auth
```



#### 获取系统信息

```http
GET /info
```

相当于 `docker system info`



#### 获取版本信息

```http
GET /version
```

相当于 `docker docker version`



#### ping

```
GET /_ping
```

示例：

```bash
$ curl http://127.0.0.1:2376/_ping
OK
```



#### 查看事件列表

```http
GET /events
```

相当于 `docker system events`



#### 获取数据使用信息

```http
GET /system/df
```

相当于 `docker system df`



#### 