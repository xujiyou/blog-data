# Redis 键过期

例子:

设置键

```
> SET mykey "Hello"
```

设置键过期时间：

```
> EXPIRE mykey 10
```

查看键的过期时间：

```
> TTL mykey
```

时间是秒，当超过了过期时间时，会自动删除 key。

通过重新设置 key ，可以自动删掉过期时间，比如：

```
> SET mykey "Hello"
> EXPIRE mykey 100
> TTL mykey
> SET mykey "Hello World"
> TTL mykey
```

如果重新设置了 key ，则 key 的 TTL 会变为 -1。

默认所有的 key，如果没设置过期时间，则它的 TTL 都会是 -1。如果 key 过期了，则它的 TTL 会是 -2。

也可以通过 PERSIST 命令来清除过期时间：

```
> PERSIST mykey
```

