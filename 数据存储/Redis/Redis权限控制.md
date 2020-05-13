# Redis 权限控制

官方文档：https://redis.io/topics/acl

权限控制是 Redis 6.0 中新加入的功能。

Redis 访问控制列表 (ACL)，是一项可以实现限制特定客户端连接可执行命令和键访问的功能，它的工作方式是：客户端连接服务器以后，客户端需要提供用户名和密码用于验证身份：如果验证成功，客户端连接会关联特定用户以及用户相应的权限。Redis 可以配置新的客户端连接自动使用 default 用户进行验证（默认选项），因此配置 default 用户权限 会使没有验证的客户端只能使用一小部分功能。Redis 6 版本 (第一个支持 ACL 的版本) 的默认配置和之前版本完全相同，即每一个新的客户端连接有权限去访问所用命令和键，因此 ACL 功能对旧版客户端和应用是向后兼容 (backward compatible) 的，并且对旧版配置用户密码的方式，使用 requirepass 配置选项是完全支持的，但是不同的是 requirepass 配置选项只是设定 default 用户的密码。RedisAUTH 命令在 Redis 6 版本进行扩展，现在可以使用两个参数形式：

```
AUTH <username><password>
```

如果使用旧版本的使用方式：

```
AUTH <password>
```

这会使用 default 用户进行验证，所以用这种形式进行验证意味着我们想使用 default 用户进行身份验证，这种方式可以提供完美的向后兼容旧版本 Redis 的支持。



可以使用下面的命令来查看 ACL 规则列表：

```
> ACL LIST 
1) "user default on nopass ~* +@all"
```

上面的命令遵循和 Redis 配置文件同样的格式返回当前 ACL 配置的规则。每一行最前面的两个词是”user”和用户名。之后的词是具体 ACL 定义的规则。我们下面将会看到怎样使用这些规则，但是现在，可以简单理解为默认（default）用户的配置是开启的（active（on）），不需要密码（require no password（nopass）），可以访问所有键（to access every possiblekey（～*)) 和可以执行任何命令（call everypossible command（+@all））的。另外，对于默认（default）用户来说，没有配置密码规则意味着新的客户端连接会自动使用默认（default）进行验证而不用显式的执行 AUTH 命令。



ACL 规则还是有些复杂的，一会再学。先来看下怎么定义用户。

```
> ACL SETUSER alice
OK
```

再次查看会看到：

```
> ACL LIST
1) "user alice off -@all"
2) "user default on nopass ~* +@all"
```

刚刚创建的 alice 用户状态是 Off 关闭状态，意思是用户被禁用，AUTH 命令将不会起作用。
不能使用任何命令，注意默认创建的用户没有使用任何命令的权限，所以 -@all 在上面的输出可以忽略，但是 ACL LIST 会用更显式的方式来告诉用户。

- 最后用户不能访问任何键模式。
- 用户没有任何密码设置。
  这样的用户是完全没有用的。让我们来定义一个开启的，设置密码的，只能访问 GET 命令并只能访问 cached：键模式的用户。

```
> ACL SETUSER alice on >p1pp0 ~cached:* +get
OK
```

然后来使用这个用户：

```
> AUTH alice p1pp0
OK
127.0.0.1:6379> GET foo
(error) NOPERM this user has no permissions to access one of the keys used as arguments
127.0.0.1:6379> GET cached
(error) NOPERM this user has no permissions to access one of the keys used as arguments
127.0.0.1:6379> GET cached:123
(nil)
127.0.0.1:6379> SET cached:123 zap
(error) NOPERM this user has no permissions to run the 'set' command or its subcommand
```

可以看到符合预期。

`ACL LIST` 命令对人不太友好，可以使用下面命令查看用户的详细信息：

```
> ACL GETUSER alice
1) "flags"
2) 1) "on"
3) "passwords"
4) 1) "2d9c75273d72b32df726fb545c8a4edc719f0a95a6fd993950b10c474ad9c927"
5) "commands"
6) "-@all +get"
7) "keys"
8) 1) "cached:*"
```

为 alice 用户添加更多的键：

```
> ACL SETUSER alice ~objects:* ~items:* ~public:*
OK
> ACL LIST
1) "user alice on #2d9c75273d72b32df726fb545c8a4edc719f0a95a6fd993950b10c474ad9c927 ~cached:* ~objects:* ~items:* ~public:* -@all +get"
2) "user default on nopass ~* +@all"
```

添加 set 权限：

```
> ACL SETUSER alice +set
OK
127.0.0.1:6379> ACL LIST
1) "user alice on #2d9c75273d72b32df726fb545c8a4edc719f0a95a6fd993950b10c474ad9c927 ~cached:* ~objects:* ~items:* ~public:* -@all +set +get"
2) "user default on nopass ~* +@all"
127.0.0.1:6379> auth alice p1pp0
OK
127.0.0.1:6379> set cached:123 world
OK
127.0.0.1:6379> get cached:123
"world"
```

切换回 default 用户：

```
> AUTH default ''
```

删除用户：

```
> ACL DELUSER alice
```











