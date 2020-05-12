# Redis 主从复制

主从复制有诸多好处与原理，这里不讲那么多，直接实操。

官方教程：https://redis.io/topics/replication

slaveof 配置已过期，改用 replicaof。

在从服务器上修改如下配置：

```
replicaof fueltank-2 6379
```

在主服务器执行：

```bash
$ redis-cli set hello world
```

在从服务器执行：

```bash
$ redis-cli --scan
```

会发现有 hello 那个键了。