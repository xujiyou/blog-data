# Docker API (六) - Exec

#### 创建一个 exec 实例

```http
POST /containers/{id}/exec
```

例子：

```bash
$ curl -L http://127.0.0.1:2376/containers/f944be090aab/exec -X POST -d '{"AttachStdin": false,"AttachStdout": true,"AttachStderr": true,"DetachKeys": "ctrl-p,ctrl-q","Tty": false,"Cmd": ["date"],"Env": ["FOO=bar","BAZ=quux"]}' -H "Content-Type: application/json"
```

返回：

```
{"Id":"1a154b4e2b7fa95e0cc457e4987baf5e3fa7d00735290957c14c06b4024f2224"}
```



#### 开始 exec

```http
POST /exec/{id}/start
```

这里的 id 就是上一步返回的 id。

例子：

```bash
curl -L http://127.0.0.1:2376/exec/1a154b4e2b7fa95e0cc457e4987baf5e3fa7d00735290957c14c06b4024f2224/start -X POST -d '{"Detach": false, "Tty": false}' -H "Content-Type: application/json"
```



#### Resize an exec instance

```http
POST /exec/{id}/resize
```

这个 API 个人感觉没卵用



#### 查看 exec 的详细信息

```http
GET /exec/{id}/json
```

