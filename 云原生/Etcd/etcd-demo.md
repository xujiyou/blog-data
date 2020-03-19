# etcd demo

首先需要先有一个 etcd 集群，这里官方有讲：https://etcd.io/docs/v3.4.0/demo/

也可以参考我的这篇文章的 etcd 部分： [Kubernetes二进制安装.md](../Kubernetes/Kubernetes二进制安装.md) 

我已经有一个 etcd 集群了，下面跟着官方教程动动手。

查看 etcd 成员：

````
$ etcdctl member list
````



`put` 写入一些数据：

```
$ etcdctl put foo "Hello World"
```



`get` 读取数据

```
$ etcdctl get foo
$ etcdctl get foo --write-out="json"
```



通过 `--profix` 获取数据

```
$ etcdctl put web1 value1
$ etcdctl put web2 value2
$ etcdctl put web3 value3
$ etcdctl get web --prefix
```



`del` 删除数据：

```
$ etcdctl del foo
$ etcdctl del web --prefix
```



`txn` 开启事务，保证原子性，这个命令很特别：

```bash
$ etcdctl put user1 bad
$ etcdctl txn --interactive
compares:
value("user1") = "bad"

success requests (get, put, del):
put result 1

failure requests (get, put, del):
put result false

SUCCESS

OK
[admin@fueltank-1 ~]$ etcdctl get result
result
1
```

解释一下：

1. etcdctl put flag 1设置flag为1
2. etcdctl txn -i开启事务（-i表示交互模式）
3. 第2步输入命令后回车，终端显示出compares：
4. 输入value("flag") = "1"，此命令是比较flag的值与1是否相等
5. 第4步完成后输入回车，终端会换行显示，此时可以继续输入判断条件（前面说过事务由条件列表组成），再次输入回车表示判断条件输入完毕
6. 第5步连续输入两个回车后，终端显示出success requests (get, put, delete):，表示下面输入判断条件为真时要执行的命令
7. 与输入判断条件相同，连续两个回车表示成功时的执行列表输入完成
8. 终端显示failure requests (get, put, delete):后输入条件判断失败时的执行列表
9. 为了看起来简洁，此实例中条件列表和执行列表只写了一行命令，实际可以输入多行
10. 总结上面的事务，要做的事情就是flag为1时设置result为true，否则设置result为false
11. 事务执行完成后查看result值为true



`watch`

```
$ etcdctl watch result
```

另外启动命令行：

```
etcdctl put result 1
```



`lease` 租约

etcd也能为key设置超时时间，但与redis不同，etcd需要先创建lease，然后使用put命令加上参数–lease=<lease ID>来设置

```
$ etcdctl lease grant 300
lease 3e2270d18d3bb8ce granted with TTL(300s)
$ etcdctl put sample value --lease=3e2270d18d3bb8ce
$ etcdctl get sample
$ etcdctl lease keep-alive 3e2270d18d3bb8ce
$ etcdctl lease revoke 3e2270d18d3bb8ce
$ etcdctl get sample
```

`lease grant `
创建lease，返回lease ID。创建的lease生存时间大于或等于ttl秒（TODO：为什么可能大于？）
`lease revoke `
删除lease，并删除所有关联的key
`lease timetolive `
取得lease的总时间和剩余时间
`lease keep-alive `
此命令不会只更新一次lease时间，而是周期性地刷新，保证它不会过期。



`lock` 分布式锁

```
$ etcdctl --endpoints=$ENDPOINTS lock mutex1

# another client with the same name blocks
$ etcdctl --endpoints=$ENDPOINTS lock mutex1
```



`elect` 用于 leader 选举

```
$ etcdctl --endpoints=$ENDPOINTS elect one p1

# another client with the same name blocks
$ etcdctl --endpoints=$ENDPOINTS elect one p2
```



查看集群状态

```
 $ etcdctl endpoint status --write-out=table
```



`snapshot` 用于操作快照

```
$ etcdctl snapshot save my.db
$ etcdctl snapshot status my.db --write-out=table
```



`migrate` 用于转换 v2 和 v3 版本的数据

```bash
# write key in etcd version 2 store
$ export ETCDCTL_API=2
$ etcdctl set foo bar

# read key in etcd v2
$ etcdctl --output="json" get foo

# stop etcd node to migrate, one by one

# migrate v2 data
$ export ETCDCTL_API=3
$ etcdctl migrate --data-dir="default.etcd" --wal-dir="default.etcd/member/wal"

# restart etcd node after migrate, one by one

# confirm that the key got migrated
$ etcdctl get /foo
```



`member` 用于增加，删除，更新 etcd 成员



`auth`,`user`,`role` 用于认证。

```bash
$ etcdctl role add root
$ etcdctl role grant-permission root readwrite foo
$ etcdctl role get root

$ etcdctl user add root
$ etcdctl user grant-role root root
$ etcdctl user get root

$ etcdctl auth enable

$ #123456 是我刚才设置的密码
$ etcdctl --user=root:123456 put foo bar
$ etcdctl get foo
$ etcdctl --user=root:123456 get foo
```



## Kubernetes 相关

查询所有 key ：

```bash
$ etcdctl get / --prefix --keys-only
```

查看所有命名空间的 key ：

```bash
$ etcdctl get /registry/namespaces --prefix --keys-only
```

