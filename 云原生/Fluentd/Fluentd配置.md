# Fluentd 配置

官方文档地址：https://docs.fluentd.org/configuration

Fluentd 的配置文件中包含以下指令：

1. **source** 输入源

2. **match** 输出目的地

3. **filter** 过滤器

4. **system** 设置配置范围

5. **label** 将 filter 和输出分组

6. **@include** 包含其他文件

下面就让我们一步一步的生成一个配置文件。



## source

source 用来定义数据来源，例子：

```
# Receive events from 24224/tcp
# This is used by log forwarding and the fluent-cat command
<source>
  @type forward
  port 24224
</source>

# http://this.host:9880/myapp.access?json={"event":"data"}
<source>
  @type http
  port 9880
</source>
```

每个 source 都要指定一个 @type 参数，指定要使用的输入插件



## match

用于指定数据的去处

```
<match myapp.access>
  @type file
  path /tmp/fluentd/access
</match>
```

当匹配到 myapp.access 之后，写入到 /tmp/fluentd/access 目录中



## filter

```
<filter myapp.access>
  @type record_transformer
  <record>
    host_param "#{Socket.gethostname}"
  </record>
</filter>
```

这会往源数据里加入 host_param 字段。

比如执行：

```bash
$ curl -X POST -d 'json={"event":"data"}' http://localhost:8888/myapp.access
```

会产生以下输出：

```
2020-02-28T15:41:59+08:00       myapp.access    {"event":"data","host_param":"XuJiyou.local"}
```

注意，filter 要放在 match 前面，不然不起作用！！！



## system

有以下配置可以获取：

- log_level

- suppress_repeated_stacktrace

- emit_error_log_interval

- suppress_config_dump

- without_source

- process_name (only available in *system* directive. No fluentd option)

例子，运行以下的代码后，fluentd 就不起作用了，因为有 without_source 。

```
<system>
  # equal to -qq option
  log_level error
  # equal to --without-source option
  without_source
  # ...
</system>
```

设置进程名称：

```
<system>
  process_name fluentd1
</system>
```

验证：

```bash
$ ps aux | grep fluentd1
jiyouxu          34175   0.0  0.0  4268212    576 s001  S+    4:28PM   0:00.00 grep fluentd1
root             34074   0.0  0.2  4382240  30108   ??  S     4:28PM   0:00.53 worker:fluentd1      
root             34073   0.0  0.2  4361220  26468   ??  Ss    4:28PM   0:00.43 supervisor:fluentd1 
```



## label

label 用于给 filter 和 match 分组。

比如以下配置：

```
<source>
  @type forward
</source>

<source>
  @type tail
  @label @SYSTEM
</source>

<filter access.**>
  @type record_transformer
  <record>
    # ...
  </record>
</filter>
<match **>
  @type elasticsearch
  # ...
</match>

<label @SYSTEM>
  <filter var.log.middleware.**>
    @type grep
    # ...
  </filter>
  <match **>
    @type s3
    # ...
  </match>
</label>
```

