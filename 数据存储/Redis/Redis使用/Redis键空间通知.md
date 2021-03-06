# Redis 键空间通知

参考：https://www.cnblogs.com/tinywan/p/5903988.html

参考：https://redis.io/topics/notifications



## 需求分析

1. 设置了生存时间的Key，在过期时能不能有所提示？
2. 如果能对过期Key有个监听，如何对过期Key进行一个回调处理？
3. 如何使用 Redis 来实现定时任务？

本文所说的定时任务或者说计划任务并不是很多人想象中的那样，比如说每天凌晨三点自动运行起来跑一个脚本。这种都已经烂大街了，随便一个 Crontab 就能搞定了。

​    这里所说的定时任务可以说是计时器任务，比如说用户触发了某个动作，那么从这个点开始过二十四小时我们要对这个动作做点什么。那么如果有 1000 个用户触发了这个动作，就会有 1000 个定时任务。于是这就不是 Cron 范畴里面的内容了。

​    举个最简单的例子，一个用户推荐了另一个用户，我们定一个二十四小时之后的任务，看看被推荐的用户有没有来注册，如果没注册就给他搞一条短信过去。



## Redis 实现

在 Redis 的 2.8.0 版本之后，其推出了一个新的特性——键空间消息（Redis Keyspace Notifications），它配合 2.0.0 版本之后的 SUBSCRIBE 就能完成这个定时任务

在 Redis 里面有一些事件，比如键到期、键被删除等。然后我们可以通过配置一些东西来让 Redis 一旦触发这些事件的时候就往特定的 Channel 推一条消息。

大致的流程就是我们给 Redis 的某一个 db 设置过期事件，使其键一旦过期就会往特定频道推消息，我在自己的客户端这边就一直消费这个频道就好了。

以后一来一条定时任务，我们就把这个任务状态压缩成一个键，并且过期时间为距这个任务执行的时间差。那么当键一旦到期，就到了任务该执行的时间，Redis 自然会把过期消息推去，我们的客户端就能接收到了。这样一来就起到了定时任务的作用。



需要修改配置：

```
notify-keyspace-events "Ex"
```

重启 Redis。

然后在一个命令行里面订阅过期命令：

```
> PSUBSCRIBE __keyevent@0__:expired
```

然后在另外一个窗口创建一个 20 秒后过期的键：

```
> SETEX mykey 20 myvalue
```

查看剩余时间：

````
> ttl mykey
````

等到键过期后，就会看到第一个命令行窗口里面有消息了。









