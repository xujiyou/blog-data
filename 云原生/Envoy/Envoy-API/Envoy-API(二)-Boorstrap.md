# Envoy - API (二)  -  Boorstrap

官方文档地址：https://www.envoyproxy.io/docs/envoy/latest/api-v2/bootstrap/bootstrap

## Bootstrap

Bootstrap 是所有配置的顶级配置。在启动指定配置文件时：

```bash
$ envoy -c <path to config>.{json,yaml,pb,pb_text}
```

这个配置文件中的顶级配置就是 Bootstrap API。配置大概如下：

```json
{
  "node": "{...}",
  "static_resources": "{...}",
  "dynamic_resources": "{...}",
  "cluster_manager": "{...}",
  "hds_config": "{...}",
  "flags_path": "...",
  "stats_sinks": [],
  "stats_config": "{...}",
  "stats_flush_interval": "{...}",
  "watchdog": "{...}",
  "tracing": "{...}",
  "runtime": "{...}",
  "layered_runtime": "{...}",
  "admin": "{...}",
  "overload_manager": "{...}",
  "enable_dispatcher_stats": "...",
  "header_prefix": "...",
  "stats_server_version_override": "{...}",
  "use_tcp_for_dns_lookups": "..."
}
```

其中，`node`、`static_resources` 、`dynamic_resources` 、`admin` 是之前遇到的，比较熟悉的，剩下的不熟悉的就需要学习了。

其中大部分在后面的文章学习。下面先学习下 `stats` 



---



## Stats

统计。

Envoy的主要目标之一是使网络易于理解。Envoy会根据其配置方式发出大量统计信息。通常，统计信息分为三类：

- **下游**：下游统计信息与传入的连接/请求有关。它们由侦听器，HTTP连接管理器，TCP代理过滤器等发出。
- **上游**：上游统计信息与传出连接/请求有关。它们由连接池，路由器过滤器，TCP代理过滤器等发出。
- **服务器**：服务器统计信息描述Envoy服务器实例的工作方式。诸如服务器正常运行时间或已分配内存量之类的统计信息在这里分类。

从v2 API开始，Envoy能够支持自定义可插入接收器。Envoy中包含一些标准接收器实现，就是这里的 `Stats`。这也是 Istio 取消 Mixer 的原因，因为统计数据的功能已经部署在 Envoy 中了。

Envoy发出三种类型的值作为统计信息：

- **计数器**：只会增加而不会减少的无符号整数。例如，总请求数。
- **量规**：递增和递减的无符号整数。例如，当前活动的请求。
- **直方图**：无符号整数，它们是值流的一部分，然后由收集器汇总以最终产生汇总的百分位值。例如，上游请求时间。

在内部，对计数器和量规进行批处理并定期冲洗以提高性能。直方图在接收时被写入。

在 Boorstrap 中，包含三个相关配置：

```json
 "stats_sinks": [],
 "stats_config": "{...}",
 "stats_flush_interval": "{...}",
```



---



## StatsSink

StatsSink 用来配置接收器。

在上面 Boorstrap 中的定义为 ：

```json
{
	"stats_sinks": [],
}
```

Envoy 内置了几种 Stats Sink，分别是：

- envoy.statsd
- envoy.dog_statsd
- envoy.metrics_service
- envoy.stat_sinks.hystrix

分别对应不同的收集、展示系统。

下面来动手搭一套 envoy.statsd  ---> statsd exporter  ---> prometheus  --->  Grafana 的系统。

下面从头到尾搭建这套系统，使用  [Envoy二进制构建及安装.md](../Envoy二进制构建及安装.md)  中搭建的环境。

StatsSink 的具体配置如下：

```json
{
  "name": "...",
  "config": "{...}",
  "typed_config": "{...}"
}
```

其中，config 和 typed_config 是互斥的。



---



## 搭建 Envoy

开始动手。。。

创建一个项目，名为 stats，所有文件都在这个项目之下。

创建 Envoy 的配置文件，文件名是 envoy-config.yaml ，内容如下：

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

stats_sinks:
  - name: envoy.statsd
    config:
      tcp_cluster_name: statsd-exporter
      prefix: hello-service

static_resources:
  listeners:
  - address:
      socket_address:
        address: 0.0.0.0
        port_value: 82
    filter_chains:
    - filters:
      - name: envoy.filters.network.http_connection_manager
        typed_config:
          "@type": type.googleapis.com/envoy.config.filter.network.http_connection_manager.v2.HttpConnectionManager
          codec_type: auto
          stat_prefix: ingress_http
          access_log:
            name: envoy.file_access_log
            typed_config:
              "@type": type.googleapis.com/envoy.config.accesslog.v2.FileAccessLog
              path: /dev/stdout
          route_config:
            name: local_route
            virtual_hosts:
            - name: service
              domains:
              - "*"
              routes:
              - match:
                  prefix: "/hello"
                route:
                  cluster: hello-service
          http_filters:
          - name: envoy.filters.http.router
  clusters:
  - name: hello-service
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    load_assignment:
      cluster_name: hello-service
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: 127.0.0.1
                port_value: 8082
  - name: statsd-exporter
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: ROUND_ROBIN
    hosts:
    - socket_address:
        address: 127.0.0.1
        port_value: 9125
```

这里使用静态配置，简单，直观，方便理解。

注意这里定义了 `stats_sinks` ，并指向一个 cluster，这个名为 statsd-exporter 的 cluster 后面部署。

下面来搭建后台服务，很简单，就是一个 go 语言实现的 HTTP 服务，文件名为 main.go，具体代码如下：

```go
package main

import (
    "github.com/emicklei/go-restful"
    "io"
    "log"
    "net/http"
)

// This example shows the minimal code needed to get a restful.WebService working.
//
// GET http://localhost:8082/hello

func main() {
    ws := new(restful.WebService)
    ws.Route(ws.GET("/hello").To(hello))
    restful.Add(ws)
    log.Println("server start in 8082")
    log.Fatal(http.ListenAndServe(":8082", nil))
}

func hello(req *restful.Request, resp *restful.Response) {
    log.Println("request hello")
    _,_ = io.WriteString(resp, "world")
}
```

然后依次在一个新命令行中执行：

```bash
$ go mod init stats
$ go build
$ ./stats
```

这样就把服务启动起来了。

这样配置好之后，在访问 Envoy 时，会形成 statsd 格式的统计信息，并会把信息发送给 statsd-exporter，后面来部署  statsd-exporter。

然后在本地命令行中启动 Envoy：

```bash
$ sudo getenvoy run standard:1.14.1 -- --config-path ./envoy-config.yaml
```

在另外一个新命令行中测试访问：

```bash
$ curl http://localhost:82/hello
world
```

成功，说明配置都没出错，下面来部署 statsd-exporter



---



## 部署  statsd-exporter

stated-exporter github 仓库：https://github.com/prometheus/statsd_exporter

我这里使用 docker-compose 来部署。

先编写 `statsd_mapping.yml` 文件，我也不会配置，只写一行就行了：

```yaml
mappings:
```

然后修改 docker-compose：

```yaml
version: "3.7"
services:

  statsd-exporter:
    image: prom/statsd-exporter
    volumes:
      - ./statsd_mapping.yml:/etc/statsd_mapping.yml
    environment:
      STATSD.MAPPING-CONFIG: /etc/statsd_mapping.yml
    expose:
      - "9125"
      - "9102"
    ports:
      - "9125:9125"
      - "9102:9102"
```

启动容器：

```bash
$ sudo docker-compose up --build -d
```

这时候看下端口占用，发现在宿主机已经有端口占用了，下面来启动 Prometheus。



---



## 启动 Prometheus

安装方式见： [Prometheus入门.md](../../Prometheus/Prometheus入门.md) 

修改配置文件如下：

```yaml
global:
  scrape_interval:     15s
  evaluation_interval: 15s
scrape_configs:
  - job_name: 'statsd'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9102']
        labels:
          group: 'services'
```

启动 prometheus：

```bash
$ ./prometheus --config.file=prometheus.yml
```

启动完成后，访问几下 envoy，再等上 15 秒，就可以在 Prometheus 中看到数据了！！！

可以通过访问 `http://fueltank-1:9090/graph` 来搜索。

通过下面方式来查看 statsd-exporter 的所有 metrics：

```bash
$ curl http://fueltank-1:9102/metrics
```

发现其中已经有关于 hello-service 的 metrics 了。

比如我查找访问成功了多少次：

```
hello_service_cluster_hello_service_upstream_rq_200
```

对比 go 服务打印的日志数量，发现结果挺准的！



---



## 启动 Grafana

首先，安装，编辑 `/etc/yum.repos.d/grafana.repo` 文件：

```
[grafana]
name=grafana
baseurl=https://packages.grafana.com/oss/rpm
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://packages.grafana.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
```

然后安装：

```bash
$ sudo yum install grafana -y
```

启动：

```bash
$ sudo systemctl enable grafana-server.service
$ sudo systemctl daemon-reload
$ sudo systemctl start grafana-server
$ sudo systemctl status grafana-server
```

完成后，打开： http://fueltank-1:3000 ，用户名及密码默认是 admin/admin，登录完成后需改密码，我这里改成了123456。

进入后，先添加 Prometheus 的数据源。statsd-exporter 自带了俩 Dashboard，我们也可以根据 PromQL 自定义 Dashboard，比如：

```
rate(hello_service_cluster_hello_service_upstream_rq_200[5m])
```

我自定义的图如下：

![image-20200421162913479](../../../resource/image-20200421162913479.png)

至此，就完全理解了这些过程！





