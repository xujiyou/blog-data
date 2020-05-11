# Echo 过滤器

Envoy 本身可以作为 Echo 协议的服务端！！！

下面是一个简单的配置：

```yaml
admin:
  access_log_path: /dev/stdout
  address:
    socket_address:
      address: 0.0.0.0
      port_value: 8081

node:
  cluster: hello-service
  id: node1

static_resources:
  listeners:
  - address:
      socket_address:
        address: 0.0.0.0
        port_value: 82
    filter_chains:
    - filters:
      - name: envoy.filters.network.echo
        typed_config: {}
```

启动 Envoy，然后进行测试：

```bash
$ telnet 127.0.0.1 82
```

操作日志如下：

```
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
hello
hello
nihao
nihao
^]
telnet> close
Connection closed.
```

