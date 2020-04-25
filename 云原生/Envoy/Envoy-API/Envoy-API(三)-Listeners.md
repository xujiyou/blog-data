# Envoy - API (三)  -  Listeners

配置地址：https://www.envoyproxy.io/docs/envoy/latest/configuration/listeners/listeners

API 地址：https://www.envoyproxy.io/docs/envoy/latest/api-v2/listeners/listeners



## 配置

Listener 的总体配置如下：

```yaml
{
  "name": "...",
  "address": "{...}",
  "filter_chains": [],
  "use_original_dst": "{...}",
  "per_connection_buffer_limit_bytes": "{...}",
  "metadata": "{...}",
  "drain_type": "...",
  "listener_filters": [],
  "listener_filters_timeout": "{...}",
  "continue_on_listener_filters_timeout": "...",
  "transparent": "{...}",
  "freebind": "{...}",
  "socket_options": [],
  "tcp_fast_open_queue_length": "{...}",
  "traffic_direction": "...",
  "udp_listener_config": "{...}",
  "api_listener": "{...}",
  "connection_balance_config": "{...}",
  "reuse_port": "...",
  "access_log": []
}
```

其中，`address` 和 `filter_chains` 是比较熟悉的。

`name` 是 Listener 的唯一名称。如果未提供名称，则 Envoy 将为 Listener 分配内部UUID。如果要通过LDS动态更新或删除侦听器，则必须提供唯一的名称。

`address` 就是要监听的地址，形式如下：

```yaml
address:
  socket_address:
    address: 0.0.0.0
    port_value: 82
```

这种定义地址的形式会经常用到。

`filter_chains` 一个拦截器链，这个后面要重点学习。

`use_original_dst` 如果使用iptables重定向了连接，则代理在其上接收连接的端口可能与原始目标地址不同。当此标志设置为true时，侦听器将重定向到与原始目标地址关联的侦听器的重定向连接。如果没有与原始目标地址关联的侦听器，则连接由接收该侦听器的侦听器处理。默认为false。配置已过期，使用 `listener_filters` 代替

`per_connection_buffer_limit_bytes` 监听器的新连接读写缓冲区大小的软限制。如果未指定，则应用实现定义的默认值（1MiB）。

`metadata`  map 形式的元数据。

`listener_filters_timeout` 等待所有侦听器筛选器完成操作的超时。如果达到超时，除非将`continue_on_listener_filters_timeout` 设置为true，否则将关闭连接，指定0以禁用超时。如果未指定，则使用默认超时15s。

`continue_on_listener_filters_timeout` 如果为 true ，则超时也不关闭连接，默认为 false。

`transparent` 是否将侦听器设置为透明套接字。当此标志设置为true时，可以使用iptables TPROXY目标将连接重定向到侦听器，在这种情况下，原始的源地址和目标地址以及端口将保留在接受的连接上。此标志应与original_dst侦听器过滤器结合使用，以将连接的本地地址标记为“已还原”。

`freebind` 侦听器是否应设置IP_FREEBIND套接字选项。当此标志设置为false时，套接字上的选项IP_FREEBIND被禁用。如果未设置此标志（默认），则不会修改套接字，即既未启用也未禁用该选项。

`socket_options` Envoy源代码或预编译的二进制文件中可能没有的其他套接字选项。

`tcp_fast_open_queue_length`  

`udp_listener_config` 如果协议中侦听器套接字地址中的协议是UDP，则此字段指定要创建的实际udp侦听器。

`api_listener`  用于表示API侦听器，用于非代理客户端。向非代理应用程序公开的API的类型取决于API侦听器的类型。设置此字段后，除名称外，不得设置其他任何字段。目前只能安装一个ApiListener。而且只能通过bootstrap config来完成，而不能通过LDS来完成。

`connection_balance_config` 侦听器的连接平衡器配置，当前仅适用于TCP侦听器。如果未指定配置，则Envoy不会尝试平衡辅助线程之间的活动连接。

`reuse_port`  当此标志设置为true时，侦听器将设置SO_REUSEPORT套接字选项，并为每个工作线程创建一个套接字，这使得在有大量连接的情况下，入站连接在工作线程之间大致均匀地分布。当此标志设置为false时，所有辅助线程共享一个套接字。在Linux v4.19-rc1之前，热重启动期间可能会拒绝新的TCP连接

`access_log` 配置 Listener 的日志。



## 代理 UDP

官方教程：https://www.envoyproxy.io/docs/envoy/latest/configuration/listeners/udp_filters/udp_proxy

目前 udp 代理还处于 alpha 阶段。

先启动一个 UDP 服务端：

```bash
$ nc -l -u -p 1235
```

- -l ：表示 listener ，监听
- -u：UDP
- -p：指定端口

编写 envoy 的配置文件：

```yaml
admin:
  access_log_path: /tmp/admin_access.log
  address:
    socket_address:
      protocol: TCP
      address: 127.0.0.1
      port_value: 9901

static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address:
        protocol: UDP
        address: 127.0.0.1
        port_value: 1234
    reuse_port: true
    access_log:
      name: envoy.file_access_log
      typed_config:
        "@type": type.googleapis.com/envoy.config.accesslog.v2.FileAccessLog
        path: /dev/stdout
    listener_filters:
      name: envoy.filters.udp_listener.udp_proxy
      typed_config:
        '@type': type.googleapis.com/envoy.config.filter.udp.udp_proxy.v2alpha.UdpProxyConfig
        stat_prefix: service
        cluster: service_udp
  clusters:
  - name: service_udp
    connect_timeout: 0.25s
    type: STATIC
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: service_udp
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: 127.0.0.1
                port_value: 1235
```

注意这里使用到了 `listener_filters` ，这里的 `envoy.filters.udp_listener.udp_proxy` 是不可以改的。

在另外一个命令行启动：

```bash
$ sudo getenvoy run standard:1.14.1 -- --config-path ./envoy-config.yaml
```

在第三个命令行测试：

```bash
$ nc -u 127.0.0.1 1234
hello
world
```

输入一些信息之后，发现第一个命令行窗口也打印了相同的信息，说明代理成功！！！



## DNS 代理

Dns 代理也是处于 alpha 阶段。

Envoy 配置文件：

```yaml
admin:
  access_log_path: /tmp/admin_access.log
  address:
    socket_address:
      protocol: TCP
      address: 127.0.0.1
      port_value: 9901

static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address:
        protocol: UDP
        address: 127.0.0.1
        port_value: 1234
    reuse_port: true
    access_log:
      name: envoy.file_access_log
      typed_config:
        "@type": type.googleapis.com/envoy.config.accesslog.v2.FileAccessLog
        path: /dev/stdout
    listener_filters:
      name: envoy.filters.udp.dns_filter
      typed_config:
        "@type": type.googleapis.com/envoy.config.filter.udp.dns_filter.v2alpha.DnsFilterConfig
        stat_prefix: dns_filter_prefix
        server_config:
          inline_dns_table:
            external_retry_count: 3
            known_suffixes:
              - suffix: "domain1.com"
              - suffix: "domain2.com"
              - suffix: "domain3.com"
            virtual_domains:
              - name: "www.domain1.com"
                endpoint:
                  address_list:
                    address:
                      - 10.0.0.1
                      - 10.0.0.2
              - name: "www.domain2.com"
                endpoint:
                  address_list:
                    address:
                      - 2001:8a:c1::2800:7
              - name: "www.domain3.com"
                endpoint:
                  address_list:
                    address:
                      - 10.0.3.1
```

根据这个配置文件来启动 Envoy，启动之后，进行测试：

```bash
$ nslookup -port=1234 www.domain1.com 127.0.0.1
```











