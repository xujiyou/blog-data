# HTTP Runtime

官方文档：https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_conn_man/runtime

HTTP 这里，支持几个运行时参数，分别是 ：

- **http_connection_manager.normalize_path**
- **tracing.client_enabled**
- **tracing.global_enabled**
- **tracing.random_sampling**

如何修改那，可以通过以下方式：

```bash
$ curl http://fueltank-1:20001/runtime_modify?tracing.client_enabled=10 -X POST
```

要小心使用 [/runtime_modify](https://www.servicemesher.com/envoy/operations/admin.html#operations-admin-interface-runtime-modify) 端点。 其变更是立即生效的。管理接口的[妥善的保护](https://www.servicemesher.com/envoy/operations/admin.html#operations-admin-interface-security)是非常**重要**的。

通过提交的参数对运行时数值进行添加或修改。要删除一个之前加入的键，只需要使用一个空值即可。注意这种删除操作，只适用于这一端点中使用重载方式加入的值；从磁盘中载入的值是能通过重载进行修改，无法删除。

修改完成后，查看修改后的结果：

```bash
$ curl http://fueltank-1:20001/runtime
```

响应如下：

```yaml
{
    "layers": [
        "base",
        "admin"
    ],
    "entries": {
        "tracing.client_enabled": {
            "final_value": "10",
            "layer_values": [
                "",
                "10"
            ]
        }
    }
}
```

发现修改已经生效了。

也可以静态指定：

```yaml
layered_runtime:
  layers:
    - name: my-runtime
      static_layer:
        health_check:
          min_interval: 5
        tracing:
          client_enabled: 20
```

或者通过 RTDS 指定：

```yaml
layered_runtime:
  layers:
    - name: my-runtime
      rtds_layer:
        name: my-runtime
        rtds_config:
          api_config_source:
            api_type: GRPC
            grpc_services:
            - envoy_grpc:
                cluster_name: xds_cluster
```

