# CoreDNS 插件

官方一共 44 个插件。

## ACL

解决的问题：**在源ip上实施访问控制策略，并防止未经授权访问DNS服务器**

阻止所有A类型记录来源为192.168.0.0/16的DNS查询：

```
. {
    acl {
        block type A net 192.168.0.0/16
    }
}
```

阻止除192.168.1.0/24之外的所有来自192.168.0.0/16的DNS查询：

```
. {
    acl {
        allow net 192.168.1.0/24
        block net 192.168.0.0/16
    }
}
```



## ANY

通过简短的HINFO答复来阻止任何查询

例子：

```
example.org {
    whoami
    any
}
```



## WHOAMI

例子：

```
example.org {
    whoami
}
```

当查询“ example.org A”时，CoreDNS将响应：

```txt
;; QUESTION SECTION:
;example.org.   IN       A

;; ADDITIONAL SECTION:
example.org.            0       IN      A       10.240.0.1
_udp.example.org.       0       IN      SRV     0 0 40212
```

该*WHOAMI*插件将每一个A或AAAA查询作出响应，不管查询名称。

如果CoreDNS在启动时找不到Corefile，则这是加载的*默认*插件。因此，它可用于检查CoreDNS是否正在响应查询。除此之外，此插件在生产中的使用受到限制。



## bind

绑定某个地址，比如：

```
. {
    bind 127.0.0.1
}
```

则只能本机可以使用。



## cache

用于设置从其他DNS服务器中获取的结果的缓存时间，比如：

```
. {
    forward . 8.8.8.8:53
    cache 10
}
```



## forward

用于转发到其他 DNS 服务器



## errors

查询处理过程中遇到的任何错误都将打印到标准输出。可以合并某些类型的错误，并每隔一段时间打印一次。

每个服务器块只能使用一次该插件。



## log

通过仅使用*日志，*您可以将所有查询（以及用于回复的部分）转储到标准输出中。存在一些选项可以稍微调整输出。请注意，对于繁忙的服务器，日志记录会导致性能下降。

启用或禁用*日志*插件仅会影响查询日志记录，无论如何，都会显示来自CoreDNS的任何其他日志记录。



## prometheus

为 CoreDNS 开启监控，可以访问 /metrics ，例子：

```
. {
    prometheus localhost:9253
}
```

这样就可以通过  http://localhost:9153/metrics 来访问指标信息了。



## hosts

使用 hosts 信息，比如：

Load `/etc/hosts` file.

```corefile
. {
    hosts
}
```

Load `example.hosts` file in the current directory.

```
. {
    hosts example.hosts
}
```

Load example.hosts file and only serve example.org and example.net from it and fall through to the next plugin if query doesn’t match.

```
. {
    hosts example.hosts example.org example.net {
        fallthrough
    }
}
```

Load hosts file inlined in Corefile.

```
example.hosts example.org {
    hosts {
        10.0.0.1 example.org
        fallthrough
    }
    whoami
}
```



## health

该插件提供健康检查点：

```
. {
    health localhost:8091
}
```

curl http://localhost:8091/health



## import

引入其他文件中的配置：

````
. {
   import config/common.conf
}
````

Where `config/common.conf` contains:

```
prometheus
errors
log
```

This imports files found in the zones directory:

```
import ../zones/*
```









