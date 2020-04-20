# Envoy 命令行选项

参考官方文档：https://www.envoyproxy.io/docs/envoy/latest/operations/cli

或者使用 `envoy -h` 查看

共 33 个选项



#### -c <path string>, --config-path <path string>

v2 版本的 [JSON/YAML/proto3 配置文件](https://www.envoyproxy.io/docs/envoy/latest/configuration/configuration#config) 路径，如果这个选项没有设置，那么就需要设置 `--config-yaml` ，这个配置文件包含了[v2 bootstrap 配置](https://www.envoyproxy.io/docs/envoy/latest/configuration/overview/bootstrap#config-overview-bootstrap) ，文件的后缀名可以是 `.json`, `.yaml`, `.pb` and `.pb_text`, 分别代表 JSON、YAML、[binary proto3](https://developers.google.com/protocol-buffers/docs/encoding) 和 [text proto3](https://developers.google.com/protocol-buffers/docs/reference/cpp/google.protobuf.text_format) 格式的文件。



#### **--config-yaml** <yaml string>

YAML 格式的 v2 bootstrap 配置字符串，这个选项中的字符串配置会覆盖 --config-path 中提供的配置。

因为 YAML 支持 JSON 格式，所以 JSON 字符串也可以作为选项值，比如：

```bash
$ envoy -c bootstrap.yaml --config-yaml "node: {id: 'node1'}"
```



#### **--mode** <string>

- `serve` ： 默认值，验证JSON配置，然后启动服务
- `validate`： 验证JSON配置，然后退出，打印“ OK”（在这种情况下，退出码为0）或配置文件生成的任何错误（退出码1），没有生成网络流量，并且不执行热重启过程，因此不会干扰计算机上的其他Envoy进程。



#### --admin-address-path <path string>

admin管理端的地址和端口将被写入的文件路径。



#### **--local-address-ip-version** <string>

用于填充服务器本地IP地址的IP地址版本，此参数影响各种标头，包括附加到X-Forwarded-For（XFF）标头的内容，可以设置为 `v4` 或 `v6`，默认是 `v4` 。



#### **--base-id** <integer>

分配共享内存区域时要使用的 base ID，Envoy在 [热重启](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/operations/hot_restart#arch-overview-hot-restart) 过程中使用共享内存区域。大多数用户将永远不必设置此选项。但是，如果Envoy需要在同一台计算机上多次运行，则每个正在运行的Envoy都将需要一个唯一的基本ID，以便共享内存区域不会发生冲突。



#### --concurrency <integer>

要运行的 [工作线程数](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/intro/threading_model#arch-overview-threading) 。如果未指定，则默认为计算机上的硬件线程数。



#### -l <string>, --log-level <string>

日志记录级别。非开发人员通常不应设置此选项。请参阅帮助文本以获取可用的日志级别和缺省值。



#### **--component-log-level** <string>

每个组件的日志记录级别的逗号分隔列表,非开发人员通常不应设置此选项。

比如：如果希望上游组件在调试级别运行，而连接组件在跟踪级别运行，则应设置 `upstream:debug,connection:trace`  

See `ALL_LOGGER_IDS` in [/source/common/common/logger.h](https://github.com/envoyproxy/envoy/blob/167df8c4554073d5115316ac36dd97088c3e6d93//source/common/common/logger.h) for a list of components.



#### **--cpuset-threads**

如果未设置--concurrency，则此标志用于控制辅助线程的数量,如果启用，则分配的cpuset大小用于确定基于Linux的系统上的工作线程数，否则，工作线程数将设置为计算机上的硬件线程数。



#### **--log-path** <path string>

日志应写入的输出文件路径。处理SIGUSR1时将重新打开此文件。如果未设置，则写入到 stderr。



#### --log-format <format string>

日志格式。



#### **--log-format-escaped**

日志相关



#### **--restart-epoch** <integer>

热重启的 epoch，Envoy已热重新启动而不是重新启动的次数，首次启动时默认为0。

此选项告诉Envoy是尝试创建热重启所需的共享内存区域，还是打开现有的共享内存区域。每次热重启时都应增加它。

[热重启包装器](https://www.envoyproxy.io/docs/envoy/latest/operations/hot_restarter#operations-hot-restarter) 设置RESTART_EPOCH环境变量，该变量在大多数情况下应传递给此选项。



#### **--hot-restart-version**

输出二进制文件的不透明热重启兼容性版本。可以将其与GET / hot_restart_version admin端点的输出进行匹配，以确定新二进制文件和正在运行的二进制文件是否与热重启兼容。



#### **--service-cluster** <string>

定义运行Envoy的本地服务集群名称，本地服务群集名称首先来自Bootstrap节点消息的cluster字段。此CLI选项提供了一种指定此值的替代方法，它将覆盖引导程序配置中设置的任何值。



#### **--service-node** <string>





#### --disable-extensions <string>

以逗号分隔的要禁用的扩展插件列表



#### **--version**

查看 Envoy 版本



