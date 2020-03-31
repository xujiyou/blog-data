# Vault Getting Start

关于安装  [Vault简介及入门.md](Vault简介及入门.md) 里面已经介绍了，dev 模式和高可用模式都有介绍，下面来学习使用。



## 操作 Secret

创建一个 Secret，在创建之前需要 enable 一个需要使用到的 path(注意这个 path 的类型是 kv)。

```bash
$ vault secrets enable -path=secret/ kv
```

创建 secret :

```bash
$ vault kv put secret/hello foo=world
```

也可以 put 多个键值对：

```bash
$ vault kv put secret/hello foo=world excited=yes
```

获取值：

```bash
$ vault kv get secret/hello
```

获取某个字段的值：

```bash
$ vault kv get -field=foo secret/hello
```

设置输出格式：

```bash
$ vault kv get -format=json secret/hello
```

获取路径列表：

```bash
$ vault kv list secret/
```

删除 Secret:

```bash
$ vault kv delete secret/hello
```



## Secrets Engines

在操作 secret 之前，必须先启用一个路径，如下面这样：

```bash
$ vault secrets enable -path=kv kv
```

这相当于 ：

````bash
$ vault secrets enable kv
````

取消启用可以这样：

```bash
$ vault secrets disable kv
```

获取路径(即引擎)列表：

```bash
$ vault secrets list
```





## 动态 Secret

与`kv`您必须自己将数据放入存储库的机密不同，动态机密在访问时会生成。在读取动态机密之前，它们不存在，因此不存在有人窃取它们或使用相同机密的其他客户端的风险。由于Vault具有内置的撤消机制，因此动态机密在使用后可以立即被吊销，从而最大程度地减少了机密存在的时间。

## path-help

查看引擎的帮助信息：

```bash
$ vault path-help kv
```

查看具体路径的帮助信息：

```bash
$ vault path-help secret/hello
```



## 认证方式

Vault 有多种认证方式，我已经测试过默认认证（token认证）和 OIDC 认证，都挺好用的。

#### token认证

```bash
$ vault token create
$ vault login s.goGkULg98E3EyaxQ57IkwMgZ
$ vault token revoke s.goGkULg98E3EyaxQ57IkwMgZ
```

#### Github 认证

```bash
$ vault auth enable -path=github github
$ vault auth list
$ vault auth help github
```





