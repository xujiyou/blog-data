# Redis 事务

事务是数据库系统的一个很关键的特性，Redis 也是支持这一特性的。

但是在 Redis Cluster 中，是不支持事务的！！！

Redis Cluster还有其他缺陷：

1. key批量操作支持有限。如mset、mget，目前只支持具有相同slot值的key执行批量操作。对弈映射为不同slot值的key由于执行mget。mset等操作可能存在多个节点上，因此不被支持。
2. key事务操作支持有限。同理只支持多key在统一节点上的事务操作，当多个key分布在不同的节点上时无法使用事务功能。
3. key作为数据分区的最小粒度读，因此不能将一个大的键值对象如hash、list等映射到不同的节点。
4. 不支持多数据空间。单机下的Redis可以支持16个数据库，集群模式下只能使用一个数据库空间，即db0.
5. 复制结构值支持一层，从节点只能复制主节点，不支持嵌套梳妆复制结构。

所以这里只能找一个单机进行测试！！！



依次执行以下命令：

```
> MULTI
> INCR foo
> INCR bar
> EXEC
```



而且，Redis 不支持回滚！！！当 Redis 在执行事务时遇到错误后，还会执行其他命令，Redis 官方认为应该让开发人员自己修复错误。



## DISCARD

执行 DISCARD，而不执行 EXEC，就会放弃掉本次事务！如下：

```
> SET foo 1
> MULTI
> INCR foo
> DISCARD
> GET foo
```



## WATCH

watch 用于原子操作，如下：

```
WATCH mykey
val = GET mykey
val = val + 1
MULTI
SET mykey $val
EXEC
```







 

