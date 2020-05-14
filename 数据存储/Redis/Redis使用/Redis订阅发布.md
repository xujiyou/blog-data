# Redis 订阅 / 发布

下面订阅两个 key：

```bash
$ redis-cli SUBSCRIBE foo bar
```

发布：

```bash
$ redis-cli PUBLISH foo foo1
$ redis-cli PUBLISH bar bar1
```

`SUBSCRIBE` 只能订阅具体的 key，不能使用通配符，如果想使用通配符，可以使用 `PSUBSCRIBE` 命令

使用通配符订阅：

```bash
$ redis-cli PSUBSCRIBE 'client:*'
```

发布：

```bash
$ redis-cli PUBLISH client:123 client-123
```

