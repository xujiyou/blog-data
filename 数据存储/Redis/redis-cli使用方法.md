# redis-cli 使用方法

官方教程：https://redis.io/topics/rediscli

redis-cli 是操作 Redis 的命令行接口。



## 常用操作

可以直接在命令行对 Redis 内的数据进行操作：

```bash
$ redis-cli incr mycounter
$ redis-cli INCR mycounter
$ redis-cli get mycounter
```

上面返回的数据显示了类型，如果不想看类型，就想看原始数据，可以这样：

```bash
$ redis-cli --raw get mycounter
$ redis-cli --raw incr mycounter
```

输出重定向：

```bash
$ redis-cli incr mycounter > /tmp/output.txt
$ cat /tmp/output.txt
5
```

指定服务端的地址：

```bash
$ redis-cli -h 127.0.0.1 -p 6379 ping
PONG
```

或者使用 uri：

```bash
$ redis-cli -u redis://123456@127.0.0.1:6379/0 ping
```





## 指定密码

指定密码，需要先为 Redis 加上密码，修改配置：

```
requirepass 123456
```

重启 Redis，然后执行：

```bash
$ redis-cli -a '123456' get mycounter
```

引号在密码中有特殊字符时需要加，比如 $，这里 a 是 auth 的简称。

也可以使用 REDISCLI_AUTH 环境变量，如下：

```bash
$ export REDISCLI_AUTH=123456
$ redis-cli get mycounter
```

有个警告说：

```
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
```

uri（u 表示 uri）或者密码放在命令行不安全，但是并没有啥安全的做法。下面这个做法还算安全：

```bash
$ set +o history
$ export PASS=123456
$ set -o history
$ redis-cli -a $PASS get mycounter 2> /dev/null
```



## 指定数据库

Redis 中数据库的概念并不是和其他数据库一样，而是用数字 0-15 代表数据库，可以用下面的命令来验证：

```bash
$ redis-cli SELECT 16
```

指定数据库：

```bash
$ redis-cli -n 1 incr a
```



## TLS

如果想加入 TLS 认证，需要修改配置：

```
port 0
tls-port 6379
tls-cert-file /etc/redis/redis.crt
tls-key-file /etc/redis/redis.key
tls-ca-cert-file /etc/redis/ca.crt
```

可以使用 openssl 生成证书，具体可以看  [Kubernetes证书生成.md](../../云原生/Kubernetes/Kubernetes认证及授权系列/Kubernetes证书生成.md) 

然后重启 redis，之后通过下面的命令访问：

```bash
$ redis-cli -a 123456 --cacert /etc/redis/ca.crt ping 2> /dev/null
```

如果想打开双向认证，可以修改如下配置：

```
tls-auth-clients yes
```

重启 Redis



## 从文件中获取信息

```bash
$ echo "hello world" > /tmp/services
$ redis-cli -x set foo < /etc/services
```

-x 参数是说从 stdio 中读取最后一个参数。

从文件中读取命令：

```bash
$ cat /tmp/commands.txt
set foo 100
incr foo
append foo xxx
get foo
$ cat /tmp/commands.txt | redis-cli
OK
(integer) 101
(integer) 6
"101xxx"
```



## 不断的执行相同的命令

```bash
$ redis-cli -r 5 incr my
```

这里 -r 表示重复执行 5 次

下面这个更常用，不断的获取当前 Redis 的统计信息：

```bash
$ redis-cli -r -1 -i 1 INFO | grep rss_human
```

INFO 指令用于获取 Redis 自身的信息，这里的 负1 表示永远不间断的执行，-i 表示间隔多少秒执行一次。



## 输出 csv

```bash
$ redis-cli lpush mylist a b c d
$ redis-cli --csv lrange mylist 0 -1
```

装入一个列表，然后取出的时候用 csv 格式，0 -1 表示从头到尾。



## 运行 lua 脚本

```bash
$ cat /tmp/script.lua
return redis.call('set',KEYS[1],ARGV[1])
$ redis-cli --eval /tmp/script.lua foo , bar
```

从 Redis 3.2 版本开始， Redis 将内置一个完整的 Lua 调试器



## 互动模式

上面运行的都是命令模式，如果不加任何命令，就进入了互动模式，如下：

````bash
$ redis-cli
127.0.0.1:6379> ping
PONG
````

如下来运行一些指令：

```
127.0.0.1:6379> select 2
OK
127.0.0.1:6379[2]> dbsize
(integer) 0
127.0.0.1:6379[2]> select 0
OK
127.0.0.1:6379> dbsize
(integer) 4
```

在互动模式中，按 tab 键可以实现命令提示与切换。

在互动模式中，可以重新连接其他实例：

```
connect fueltank-1 6379
```

在互动模式中，可以查看帮助，如下：

```
help get
```



## 查看 Redis 状态

```bash
$ redis-cli --stat
```



## 查看大的键

```bash
$ redis-cli --bigkeys
```



## 查看键列表

```bash
$ redis-cli --scan | head -10
```

指定键的通配符：

```bash
$ redis-cli --scan --pattern '*m*'
```



## 订阅 / 发布

订阅以 aa 开头的键：

```bash
$ redis-cli psubscribe 'aa*'
```

然后在另一个命令行内发布：

```bash
$ redis-cli PUBLISH aaa bbb
```

这样会在订阅的命令行显示输出

如果发布如下：

```bash
$ redis-cli PUBLISH ccc ddd
```

则不会在订阅的命令行中输出。



## 监控

在一个命令行中监控：

```bash
$ redis-cli monitor
```

在另外一个命令行中执行：

```bash
$ redis-cli set a b
```

会看到监控命令行中有日志显示。



## 命令延迟

CLI使用 --latency 选项运行一个循环，在该循环中，将[PING](https://redis.io/commands/ping)命令发送到Redis实例，并测量获得答复的时间。每秒发生100次，并且统计信息会在控制台中实时更新，可用于研究Redis实例的延迟并了解延迟的最大值，平均值和分布：

```bash
$ redis-cli --latency
```

统计信息以毫秒为单位

使用下面的命令会每隔15秒记录一次，可以使用 -i 参数指定间隔时间：

```bash
$ redis-cli --latency-history
```

使用下面的命令，显示的更花里胡哨一些：

```bash
$ redis-cli --latency-dist
```

或者：

```bash
$ redis-cli --intrinsic-latency 5
```



## 备份

```bash
$ redis-cli --rdb /tmp/dump.rdb
```

查看命令结果：

```bash
$ echo $?
0
```

应该为 0，如果不为 0 则说明出错。

恢复方法:

先查看储存位置：

```bash
$ redis-cli CONFIG GET dir
```

获取到目录后，关闭服务，然后把文件放进对应的目录，最后启动 redis 即可。







