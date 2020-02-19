# 设置并运行 Logstash

参考官方文档：[Setting Up and Running Logstash](https://www.elastic.co/guide/en/logstash/7.5/setup-logstash.html)

## 文件布局

首先来看 Logstash 安装完成后的文件布局，主要来看通过 YUM/RPM 方式来安装的，因为线上主要通过这种方式来安装：

| Type         | 描述                                                         | 默认路径                    | 设置属性(默认在logstash.yml中)       |
| ------------ | ------------------------------------------------------------ | --------------------------- | ------------------------------------ |
| **home**     | Logstash 安装目录                                            | `/usr/share/logstash`       |                                      |
| **bin**      | 存放二进制可执行文件                                         | /usr/share/logstash/bin     |                                      |
| **settings** | 配置文件，包括`logstash.yml`, `jvm.options`, and `startup.options` | /etc/logstash               | path.settings                        |
| **conf**     | Logstash 运行中使用的管道配置文件                            | /etc/logstash/conf.d/*.conf | 在/etc/logstash/pipelines.yml 中配置 |
| **logs**     | Logstash 自身的日志文件                                      | /var/log/logstash           | path.logs                            |
| **plugins**  | 存放插件的地方                                               | /usr/share/logstash/plugins | path.plugins                         |
| **data**     | Logstash 及其插件需要存放永久性数据的地方                    | /var/lib/logstash           | path.data                            |

## 配置文件介绍

Logstash 有两类配置文件，第一类是管道配置文件，它定义了 Logstash 中数据的走向，另一类是 Logstash 自身的配置文件，它定义了 Logstash 该怎样运行。

### 管道配置文件

Logstash 默认会启用 `/etc/logstash/conf.d` 目录下以 `.conf` 结尾的配置文件，后面再详细讲解这类文件

### Logstash 设置文件

Logstash 包含以下设置文件：

- **`logstash.yml`** 包含了 Logstash 的主要配置，在命令行中出现的配置都可以写在这里面，不过命令行的优先级比这个文件要高。
- **`pipelines.yml`** 定义了 Logstash  要加载哪些管道配置文件。
- **`jvm.options`** 定义了一些 JVM 的选项
- **`log4j2.properties`** Logstash 自身的日志配置
- **`startup.options`** 启动配置，在 systemctl start 时使用。

## logstash.yml

| setting                      | 描述                                                         | 默认值                                 |
| ---------------------------- | ------------------------------------------------------------ | -------------------------------------- |
| node.name                    | 节点名称                                                     | 主机名                                 |
| path.data                    | 数据存放路径                                                 | LOGSTASH_HOME/data                     |
| pipeline.id                  | 管道ID，对应 /etc/logstash/pipelines.yml 中的pipeline.id     | main                                   |
| pipeline.java_execution      | 使用java执行引擎                                             | true                                   |
| pipeline.workers             | 管道工作线程数量                                             | CPU的核心数量                          |
| pipeline.batch.size          | 单个处理的事件数量，越大效率越高，但内存也越大               | 125                                    |
| pipeline.batch.delay         | 事件执行的延迟时间                                           | 50                                     |
| pipeline.unsafe_shutdown     | 为true时，关闭 Logstash 会立即退出，为 false 时，会处理完所有事件再退出。 | false                                  |
| pipeline.plugin_classloaders | (Beta) 在独立的加载器中加载插件以隔离他们的依赖关系          | false                                  |
| path.config                  | 主管道的配置文件，启动时就加载                               | Centos下是 /etc/logstash/conf.d/*.conf |
| config.string                | 主管道配置的字符串版本                                       | None                                   |
| config.test_and_exit         | 如果是 true ，Logstash 会检查配置文件并退出                  | false                                  |
| config.reload.automatic      | 若为true，则在管道配置改变时，自动重加载，接收到 SIGHUP(kill -1 pid) 信号是也会重加载 | false                                  |
| config.reload.interval       | Logstash检查配置文件更改的频率（秒）。                       | 3s                                     |
| config.debug                 | 若为true，则会打印编译和debug日志，通过设置log.level: debug也会达到这种效果 | false                                  |
| config.support_escapes       | 若为 true，则在处理字符串时会使用转译字符，如 \n等           | false                                  |
| modules                      | 插件用到的变量                                               | None                                   |
| queue.type                   | memory 或 persisted，用于事件缓冲的内部队列模型。            | memory                                 |
| path.queue                   | 若 queue.type: persisted ，则使用的储存目录                  | path.data/queue                        |
| queue.page_capacity          | 若 queue.type: persisted，分页的大小                         | 64mb                                   |
| queue.max_events             | 若 queue.type: persisted，最大的未读事件数量                 | 0 (unlimited)                          |
| queue.max_bytes              | 最大的队列总容量，应确保磁盘容量大于此处的值，如果同时指定了queue.max_events和queue.max_bytes，则Logstash将使用最先达到的条件 | 1024mb (1g)                            |
| queue.checkpoint.acks        | 启用持久队列时，检查点之前的最大事件数。为0为无限            | 1024                                   |
| queue.checkpoint.writes      | 启用持久队列时，检查点之前写入事件的最大数目                 | 1024                                   |
| queue.checkpoint.retry       | 启用后，对于任何失败的检查点写入，Logstash将在每次尝试的检查点写入时重试一次。不会重试任何后续错误。这是针对失败的检查点写入的解决方法，这些写入仅在具有非标准行为（如SAN）的文件系统上可见，除非在这些特定情况下，否则不建议使用。 | false                                  |
| queue.drain                  | 启用时，Logstash将等待持久化队列耗尽，然后关闭。             | false                                  |
| dead_letter_queue.enable     | 启用死信队列                                                 | false                                  |
| dead_letter_queue.max_bytes  | 每个死信队列的最大大小。如果条目将死信队列的大小增加到此设置之外，则将删除这些条目。 | 1024mb                                 |
| path.dead_letter_queue       | 将为死信队列存储数据文件的目录路径。                         | path.data/dead_letter_queue            |
| http.host                    | REZT API 地址                                                | "127.0.0.1"                            |
| http.port                    | REST API端口                                                 | 9600                                   |
| log.level                    | 日志等级，包括：fatal，error，warn，info，debug，trace       | info                                   |
| log.format                   | 日志格式，json 或 plain                                      | plain                                  |
| path.logs                    | 日志地址                                                     | LOGSTASH_HOME/logs                     |
| pipeline.separate_logs       | 若为true，则为每个管道生成一个单独的日志文件                 | false                                  |
| path.plugins                 | 数组，插件的地址，可以设置多个                               | yum安装的为/usr/share/logstash/plugins |

## 密钥设置

创建 keystore：

注意这里 --path.settings 是需要设置的，这会在 /etc/logstash 中生成一个 .keystore 结尾的文件

```bash
$ set +o history
$ export LOGSTASH_KEYSTORE_PASS=mypassword
$ set -o history
$ sudo -E /usr/share/logstash/bin/logstash-keystore --path.settings /etc/logstash create
```

注意，这里的 LOGSTASH_KEYSTORE_PASS 的值要与 `/etc/sysconfig/logstash` 中的值一样，这个文件需要自己创建，其内容如下：

```
LOGSTASH_KEYSTORE_PASS=mypassword
```



加入密钥

```bash
$ sudo -E /usr/share/logstash/bin/logstash-keystore --path.settings /etc/logstash add ES_PWD
```

输入密码，然后在 .keystore 文件中就生成了 ES_PWD，查看命令：

```bash
$ keytool -list -v -keystore /etc/logstash/logstash.keystore
```

然后在 Logstash 的配置文件中就可以通过 `${ES_PWD}` 来使用了，这就避免在配置文件中输入明文密码！

删除密钥

```bash
$ sudo -E /usr/share/logstash/bin/logstash-keystore --path.settings remove ES_PWD
```

## 日志

Logstash 的日志框架是基于 Log4j2 框架的，它的许多功能都直接向用户公开。

可以通过修改 **`log4j2.properties`** 文件或通过 Logstash API 来设置日志。

