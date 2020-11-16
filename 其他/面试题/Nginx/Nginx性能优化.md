# Nginx 性能优化

- Nginx 性能优化的关键点在于减少磁盘IO和网络IO。
- `http` 链接的尽快释放，减少请求的堆积。



## 调整 Work 进程数

Nginx运行工作进程个数一般设置CPU的核心或者核心数x2。

```bash
worker_processes 16; # 指定 Nginx 要开启的进程数，结尾的数字就是进程的个数，可以为 auto
```

如果想省麻烦也可以配置为`worker_processes auto;`，将由 Nginx 自行决定 worker 数量。当访问量快速增加时，Nginx 就会临时 fork 新进程来缩短系统的瞬时开销和降低服务的时间。



## 绑核

因为 Nginx 是多进程模型，每个进程中是单线程的，所以为了避免资源使用不均可以进行绑核操作。

```
worker_processes 4;
worker_cpu_affinity 0001 0010 0100 1000;
```

其中 `worker_cpu_affinity` 就是配置 Nginx 进程与 CPU 亲和力的参数，即把不同的进程分给不同的 CPU 核处理。这里的`0001 0010 0100 1000`是掩码，分别代表第1、2、3、4核CPU。上述配置会为每个进程分配一核CPU处理。

当然，如果想省麻烦也可以配置`worker_cpu_affinity auto;`，将由 Nginx 按需自动分配。



## 单个进程允许的客户端最大连接数

通过调整控制连接数的参数来调整 Nginx 单个进程允许的客户端最大连接数。

```
events {
  worker_connections 20480;
}
```

`worker_connections` 也是个事件模块指令，用于定义 Nginx 每个进程的最大连接数，默认是 1024。

最大连接数的计算公式是：`max_clients = worker_processes * worker_connections`

另外，**进程的最大连接数受 Linux 系统进程的最大打开文件数限制**，在执行操作系统命令 `ulimit -HSn 65535`或配置相应文件后， `worker_connections` 的设置才能生效。

#### 配置获取更多连接数

默认情况下，Nginx 进程只会在一个时刻接收一个新的连接，我们可以配置`multi_accept` 为 `on`，实现在一个时刻内可以接收多个新的连接，提高处理效率。该参数默认是 `off`，建议开启。

```
events {
  multi_accept on;
}
```

参考：http://nginx.org/en/docs/ngx_core_module.html#multi_accept



## Nginx 事件处理模型优化

Nginx 的连接处理机制在不同的操作系统中会采用不同的 I/O 模型，在 linux 下，Nginx 使用 epoll 的 I/O 多路复用模型，在 Freebsd 中使用 kqueue 的 I/O 多路复用模型，在 Solaris 中使用 /dev/poll 方式的 I/O 多路复用模型，在 Windows 中使用 icop，等等。

配置如下：

```
events {
  use epoll;
}
```

`events` 指令是设定 Nginx 的工作模式及连接数上限。`use`指令用来指定 Nginx 的工作模式。Nginx 支持的工作模式有 select、 poll、 kqueue、 epoll 、 rtsig 和/ dev/poll。当然，也可以不指定事件处理模型，Nginx 会自动选择最佳的事件处理模型。



## TCP 优化

```
http {
  sendfile on;
  tcp_nopush on;

  keepalive_timeout 120;
  tcp_nodelay on;
}
```

第一行的 `sendfile` 配置可以提高 Nginx 静态资源托管效率。sendfile 是一个系统调用，直接在内核空间完成文件发送，不需要先 read 再 write，没有上下文切换开销。

TCP_NOPUSH 是 FreeBSD 的一个 socket 选项，对应 Linux 的 TCP_CORK，Nginx 里统一用 `tcp_nopush` 来控制它，并且只有在启用了 `sendfile` 之后才生效。启用它之后，数据包会累计到一定大小之后才会发送，减小了额外开销，提高网络效率。

TCP_NODELAY 也是一个 socket 选项，启用后会禁用 Nagle 算法，尽快发送数据，某些情况下可以节约 200ms（Nagle 算法原理是：在发出去的数据还未被确认之前，新生成的小数据先存起来，凑满一个 MSS 或者等到收到确认后再发送）。Nginx 只会针对处于 keep-alive 状态的 TCP 连接才会启用 `tcp_nodelay`。



## 前端性能优化

Nginx 开启 gzip 压缩之后可以降低网络IO，但是对前端来说，可以提前进行 `gzip` 压缩，这样请求的时候就不用再压缩了，减少对 `cpu` 的损耗。

Nginx 给你返回静态文件的时候，会判断是否开启gzip，然后压缩后再还给浏览器。

但是nginx其实会先判断是否有.gz后缀的相同文件，有的话直接返回，nginx自己不再进行压缩处理。

压缩是要时间的！不同级别的压缩率花的时间也不一样。所以提前准备gz文件，可以更加优化。而且你可以把压缩率提高点，这样带宽消耗会更小！！！

#### vue中怎么使用

在vue中集成插件compression-webpack-plugin

**配置如下**

```js
const CompressionWebpackPlugin = require("compression-webpack-plugin"); // 开启gzip压缩， 按需引用
const productionGzipExtensions = /\.(js|css|json|txt|html|ico|svg)(\?.*)?$/i; // 开启gzip压缩， 按需写入

configureWebpack: config => {
        // 需要 npm i -D compression-webpack-plugin
        const plugins = [];
         plugins.push(new CompressionWebpackPlugin({
             filename: "[path].gz[query]",  //压缩后的文件策略
             algorithm: "gzip",             //压缩方式
             test: productionGzipExtensions,//可设置需要压缩的文件类型
        //     include:'', //符合任何这些条件的文件
        //     exclude:'',//排除压缩的文件
            threshold: 10240, //大于10240字节的文件启用压缩
            minRatio: 0.8 // 压缩比率大于等于0.8时不进行压缩
             deleteOriginalAssets: false,//是否删除压缩前的文件，默认false
        }));
        config.plugins = [...config.plugins, ...plugins];
    },
```

#### Nginx 配置

只在前端生成好gzip文件，还未能达到我们前端优化的目的。需要在服务器端开启gzip的配置才可以。

Nginx 开启 gzip 的配置：

```
server{
    gzip on;
    gzip_buffers 32 4K;
    gzip_comp_level 6;
    gzip_min_length 100;
    gzip_types application/javascript text/css text/xml;
    gzip_disable "MSIE [1-6]\."; #配置禁用gzip条件，支持正则。此处表示ie6及以下不启用gzip（因为ie低版本不支持）
    gzip_vary on;
}
```

> 不适合压缩的数据：
>
> 二进制资源：例如图片/mp3这样的二进制文件,不必压缩；因为压缩率比较小, 比如100->80字节,而且压缩也是耗费CPU资源的.



## 日志优化

可适当减少日志操作。比如访问静态资料的日志记录，如果你感觉不重要，可以取消日志记录。这样请求资源的时候就会减少日志的磁盘 io。

```bash
# 关闭日志
access_log off;
# 禁用文件未找到的错误到日志中去
log_not_found off;
```

















