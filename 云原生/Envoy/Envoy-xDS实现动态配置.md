#Envoy - xDS 实现动态配置

DS ，Discovery Service，即服务发现。xDS 表示一组 DS 的统称。具体如下：

| 简称 | 全称                         |
| ---- | ---------------------------- |
| LDS  | Listener Discovery Service   |
| RDS  | Route Discovery Service      |
| CDS  | Cluster Discovery Service    |
| EDS  | Endpoint Discovery Service   |
| ADS  | Aggregated Discovery Service |
| HDS  | Health Discovery Service     |
| SDS  | Secret Discovery Service     |
| MS   | Metric Service               |
| RLS  | Rate Limit Service           |

具体介绍请看：https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/operations/dynamic_configuration

xDS 通过 gRPC 来进行通信，在 Istio 中，Polit 组件就实现有 xDS 协议。

xDS 的目的是通过 API 来动态更新 Envoy 的配置。作为对比， Nginx 只能通过 reload 来更新静态配置。



---



## xDS API

xDS API 在envoy中被称为 `Data plane API`。其代码保存在 https://github.com/envoyproxy/envoy/tree/master/api/envoy/api/v2，用户可以根据proto文件自行生成相对应语言的GRPC代码文件。

Envoy 官方提供了两份 xDS Server 的实现，分别是：

- [go-control-plane](https://github.com/envoyproxy/go-control-plane) 基于Golang 的 xDS Server 实现代码
- [java-control-plane](https://github.com/envoyproxy/java-control-plane) 基于 Java 的 xDS Server 实现代码

另外，官方还把 api 的定义代码从 Envoy 的源码库中提取出来，放在了 https://github.com/envoyproxy/data-plane-api



---



## 构建服务端

- 参考：https://github.com/envoyproxy/go-control-plane
- 参考：https://github.com/salrashid123/envoy_control
- Envoy 版本：v1.14.1
- xDS API 版本：v2
- go-control-plane 版本：0.9.4

下来一步一步来构建一个最简单的服务端，相当于一个 hello world。

创建一个项目，名为 `my-xds`。

注意：这里有个坑，在 `go.mod` 中 `github.com/envoyproxy/go-control-plane` 不要选择 `0.9.5` 版本，HTTPGateway 有语法错误，可以选择 `0.9.4` 版本。我的 go.mod 文件如下：

```
module my-xds

go 1.14

require (
	github.com/envoyproxy/go-control-plane v0.9.4
	github.com/golang/protobuf v1.3.3
	github.com/sirupsen/logrus v1.5.0
	google.golang.org/grpc v1.28.1
)
```



下面的代码实现了 CDS 和 LDS，其他 DS 实现同理。

在  [Envoy代理自己的服务.md](Envoy代理自己的服务.md) 中，已经实现了 Envoy 的静态配置，下面我们在这个上面的基础上来实现 xDS 动态配置。（注：前端代理不要了，耽误在调试过程中发现错误）。

下面来写代码，首先写 main.go ，主要流程都在这里面：

```go
package main

import (
	"context"
	"fmt"
	v2 "github.com/envoyproxy/go-control-plane/envoy/api/v2"
	discovery "github.com/envoyproxy/go-control-plane/envoy/service/discovery/v2"
	"github.com/envoyproxy/go-control-plane/pkg/cache"
	xds "github.com/envoyproxy/go-control-plane/pkg/server"
	"google.golang.org/grpc"
	"log"
	"net"
	"net/http"
)

func main() {
	ctx := context.Background()
	log.Printf("Starting control plane")

	snapshotCache := cache.NewSnapshotCache(false, cache.IDHash{}, nil)
	snapshot := cache.NewSnapshot("1.0", nil, BuildCluster(), nil, BuildListener(), nil)
	_ = snapshotCache.SetSnapshot("node1", snapshot)

	myCallbacks := MyCallbacks{}
	srv := xds.NewServer(ctx, snapshotCache, &myCallbacks)

	RunManagementGateway(ctx, srv, 9001)
	RunManagementServer(ctx, srv, 9002)

	<-ctx.Done()
}

func RunManagementServer(ctx context.Context, server xds.Server, port uint) {
	grpcServer := grpc.NewServer()

	lis, err := net.Listen("tcp", fmt.Sprintf(":%d", port))
	if err != nil {
		log.Fatal("failed to listen: ", err)
	}

	discovery.RegisterAggregatedDiscoveryServiceServer(grpcServer, server)
	v2.RegisterEndpointDiscoveryServiceServer(grpcServer, server)
	v2.RegisterClusterDiscoveryServiceServer(grpcServer, server)
	v2.RegisterRouteDiscoveryServiceServer(grpcServer, server)
	v2.RegisterListenerDiscoveryServiceServer(grpcServer, server)

	log.Println("management server listening: ", port)
	if err = grpcServer.Serve(lis); err != nil {
		log.Fatal(err)
	}

}

func RunManagementGateway(ctx context.Context, srv xds.Server, port uint) {
	log.Println("gateway listening HTTP/1.1 :", port)
	server := &http.Server{Addr: fmt.Sprintf(":%d", port), Handler: &xds.HTTPGateway{Server: srv}}
	go func() {
		if err := server.ListenAndServe(); err != nil {
			log.Fatal(err)
		}
	}()
}
```

主要流程是：

- 先用 `BuildCluster` 和 `BuildListener` （下面有实现，参考的是上一篇文章的静态配置）来构建服务端中的信息，然后把这些信息放入缓存。
- 绑定回调函数。
- 启动 HTTP Gateway，监听 HTTP 请求，实现 REST API 。我这里监听了 9001 端口。
- 启动 gRPC Server，监听 gRPC 请求。我这里监听了 9002 端口。
- 当获取到请求后，会把缓存中的信息返回给客户端。



然后编写 cluster.go ，实现了 BuildCluster 方法，这个方法构建了 CDS 使用到的信息：

```go
package main

import (
	v2 "github.com/envoyproxy/go-control-plane/envoy/api/v2"
	core "github.com/envoyproxy/go-control-plane/envoy/api/v2/core"
	endpoint "github.com/envoyproxy/go-control-plane/envoy/api/v2/endpoint"
	"github.com/envoyproxy/go-control-plane/pkg/cache"
	"github.com/golang/protobuf/ptypes"
	"log"
	"time"
)

func BuildCluster() []cache.Resource {
	var clusterName1 = "service_bbc"
	log.Println(">>>>>>>>>>>>>>>>>>> creating cluster ", clusterName1)

	h := &core.Address{Address: &core.Address_SocketAddress{
		SocketAddress: &core.SocketAddress{
			Address:  "127.0.0.1",
			Protocol: core.SocketAddress_TCP,
			PortSpecifier: &core.SocketAddress_PortValue{
				PortValue: uint32(8080),
			},
		},
	}}

	return []cache.Resource{
		&v2.Cluster{
			Name:                 clusterName1,
			ConnectTimeout:       ptypes.DurationProto(2 * time.Second),
			ClusterDiscoveryType: &v2.Cluster_Type{Type: v2.Cluster_STRICT_DNS},
			LbPolicy:             v2.Cluster_ROUND_ROBIN,
			LoadAssignment: &v2.ClusterLoadAssignment{
				ClusterName: clusterName1,
				Endpoints: []*endpoint.LocalityLbEndpoints{
					{
						LbEndpoints: []*endpoint.LbEndpoint{
							{
								HostIdentifier: &endpoint.LbEndpoint_Endpoint{
									Endpoint: &endpoint.Endpoint{
										Address: h,
									},
								},
							},
						},
					},
				},
			},
		},
	}
}
```

这个文件很简单，就是根据官方的结构体来填数据，这里都是我们自己填的测试数据。在实际场景中，比如 Istio 中，是在 k8s apiserver 中拿数据，然后填入到这里面。并且这里也可以对接各种数据库后端来保存信息。

- 这里使用 golang 代码构建的配置和  [Envoy代理自己的服务.md](Envoy代理自己的服务.md) 中使用的静态配置是一模一样的。
- `127.0.0.1:8080` 是真正的后端服务的地址，即 hello-service（上一篇文章中构建的） 监听的地址。



再然后编写 listener.go 实现了 BuildListener，这个方法构建了 LDS 使用到的信息 ：

```go
package main

import (
	v2 "github.com/envoyproxy/go-control-plane/envoy/api/v2"
	core "github.com/envoyproxy/go-control-plane/envoy/api/v2/core"
	listener "github.com/envoyproxy/go-control-plane/envoy/api/v2/listener"
	v2route "github.com/envoyproxy/go-control-plane/envoy/api/v2/route"
	"github.com/envoyproxy/go-control-plane/pkg/cache"
	"github.com/golang/protobuf/ptypes"

	hcm "github.com/envoyproxy/go-control-plane/envoy/config/filter/network/http_connection_manager/v2"
	"log"
)

func BuildListener() []cache.Resource {
	var clusterName = "service_bbc"
	var listenerName = "listener_0"
	var targetPrefix = "/hello"
	var virtualHostName = "service"
	var routeConfigName = "local_route"

	log.Println(">>>>>>>>>>>>>>>>>>> creating listener ", listenerName)

	virtualHost := v2route.VirtualHost{
		Name:    virtualHostName,
		Domains: []string{"*"},

		Routes: []*v2route.Route{{
			Match: &v2route.RouteMatch{
				PathSpecifier: &v2route.RouteMatch_Prefix{
					Prefix: targetPrefix,
				},
			},

			Action: &v2route.Route_Route{
				Route: &v2route.RouteAction{
					ClusterSpecifier: &v2route.RouteAction_Cluster{
						Cluster: clusterName,
					},
				},
			},
		}}}

	manager := &hcm.HttpConnectionManager{
		CodecType:  hcm.HttpConnectionManager_AUTO,
		StatPrefix: "ingress_http",
		RouteSpecifier: &hcm.HttpConnectionManager_RouteConfig{
			RouteConfig: &v2.RouteConfiguration{
				Name:         routeConfigName,
				VirtualHosts: []*v2route.VirtualHost{&virtualHost},
			},
		},
		HttpFilters: []*hcm.HttpFilter{{
			Name: "envoy.filters.http.router",
		}},
	}

	pbst, err := ptypes.MarshalAny(manager)
	if err != nil {
		panic(err)
	}

	return []cache.Resource{
		&v2.Listener{
			Name: listenerName,
			Address: &core.Address{
				Address: &core.Address_SocketAddress{
					SocketAddress: &core.SocketAddress{
						Protocol: core.SocketAddress_TCP,
						Address:  "0.0.0.0",
						PortSpecifier: &core.SocketAddress_PortValue{
							PortValue: 80,
						},
					},
				},
			},
			FilterChains: []*listener.FilterChain{{
				Filters: []*listener.Filter{{
					Name: "envoy.filters.network.http_connection_manager",
					ConfigType: &listener.Filter_TypedConfig{
						TypedConfig: pbst,
					},
				}},
			}},
		}}
}
```

这个文件我调试了许久，有几处坑，学多了几处 golang 的写法。这里的 `0.0.0.0:80` ，就是 Envoy 需要监听的地址。Envoy 从这个地址监听到请求之后，就会转到 CDS 中定义的真正的后端服务。



下面创建 call_back.go，这个文件内实现了一些回调方法，代码如下：

```go
package main

import (
	"context"
	v2 "github.com/envoyproxy/go-control-plane/envoy/api/v2"
	"log"
)

type MyCallbacks struct {}

func (cb *MyCallbacks) Report() {
	log.Println("Report...")
}

func (cb *MyCallbacks) OnStreamOpen(ctx context.Context, id int64, typ string) error {
	log.Println("OnStreamOpen...")
	return nil
}

func (cb *MyCallbacks) OnStreamClosed(id int64) {
	log.Println("OnStreamClosed...")
}

func (cb *MyCallbacks) OnStreamRequest(int64, *v2.DiscoveryRequest) error {
	log.Println("OnStreamRequest...")
	return nil
}

func (cb *MyCallbacks) OnStreamResponse(int64, *v2.DiscoveryRequest, *v2.DiscoveryResponse) {
	log.Println("OnStreamResponse...")
	cb.Report()
}

func (cb *MyCallbacks) OnFetchRequest(ctx context.Context, req *v2.DiscoveryRequest) error {
	log.Println("OnFetchRequest, req:", req)
	return nil
}

func (cb *MyCallbacks) OnFetchResponse(*v2.DiscoveryRequest, *v2.DiscoveryResponse) {
	log.Println("OnFetchResponse...")
}
```

这个文件代码比较简单，就是打印日志。。。当然也可以加其他代码，比如统计信息等。这个文件的代码对调试很重要，因为看到打印日志才知道服务端接收到 Envoy 或者 HTTP 客户端的请求了。



---



## REST 访问

在自己的电脑上用 Goland 启动上面的代码：

```bash
$ go run main.go
```



在上面的代码中，启动了一个 `HTTPGateway` ，它就是用来接受 REST 请求的，方便看到效果。

关于访问方法：https://www.envoyproxy.io/docs/envoy/latest/configuration/overview/xds_api

在 `github.com/envoyproxy/go-control-plane@v0.9.4/pkg/server/gateway.go` 中也有 REST API 的地址。

通过 HTTP 来访问：

```bash
$ curl  http://localhost:9001/v2/discovery:clusters -X POST -H "Content-Type: application/json" --data '{"node": {"id": "node1"},"resourceNames": ["service_bbc"]}' | json_pp
```

响应是一串字符串，我这里进行了格式化。这里 `node.id` 是必填的，`resourceNames` 是选填的，表示要获取哪些 Cluster。

我这里的响应是：

```json 
{
   "resources" : [
      {
         "name" : "service_bbc",
         "connect_timeout" : "2s",
         "@type" : "type.googleapis.com/envoy.api.v2.Cluster",
         "load_assignment" : {
            "endpoints" : [
               {
                  "lb_endpoints" : [
                     {
                        "endpoint" : {
                           "address" : {
                              "socket_address" : {
                                 "port_value" : 8080,
                                 "address" : "127.0.0.1"
                              }
                           }
                        }
                     }
                  ]
               }
            ]
         },
         "type" : "LOGICAL_DNS",
         "dns_lookup_family" : "V4_ONLY"
      }
   ],
   "version_info" : "1.0",
   "type_url" : "type.googleapis.com/envoy.api.v2.Cluster"
}
```

并在在控制台里面，可以看到有访问日志。

访问 Listener :

```bash
$ curl  http://localhost:9001/v2/discovery:listeners -X POST -H "Content-Type: application/json" --data '{"node": {"id": "node1"}}' | json_pp
```

结果如下：

```json
{
   "version_info" : "1.0"
   "type_url" : "type.googleapis.com/envoy.api.v2.Listener",
   "resources" : [
      {
         "address" : {
            "socket_address" : {
               "port_value" : 80,
               "address" : "0.0.0.0"
            }
         },
         "@type" : "type.googleapis.com/envoy.api.v2.Listener",
         "name" : "listener_0",
         "filter_chains" : [
            {
               "filters" : [
                  {
                     "typed_config" : {
                        "stat_prefix" : "ingress_http",
                        "http_filters" : [
                           {
                              "name" : "envoy.filters.http.router"
                           }
                        ],
                        "@type" : "type.googleapis.com/envoy.config.filter.network.http_connection_manager.v2.HttpConnectionManager",
                        "route_config" : {
                           "virtual_hosts" : [
                              {
                                 "routes" : [
                                    {
                                       "route" : {
                                          "cluster" : "service_bbc"
                                       },
                                       "match" : {
                                          "prefix" : "/hello"
                                       }
                                    }
                                 ],
                                 "domains" : [
                                    "*"
                                 ],
                                 "name" : "service"
                              }
                           ],
                           "name" : "local_route"
                        }
                     },
                     "name" : "envoy.filters.network.http_connection_manager"
                  }
               ]
            }
         ]
      }
   ],
}

```

注意这个 Listeners 、它会从监听 `0.0.0.0:80` 上的请求，然后在 URL 上匹配到 `/hello` 前缀后，就转发给名为 `service_bbc` 的 Cluster，这个 Cluster 里面有定义真正的后端服务的地址。

这里通过 xDS 获取的动态配置和静态配置是一样的，只是位置有些不同。



---



## Envoy 通过 gRPC 访问 xDS 服务端

对上面的服务端代码进行交叉编译：

```bash
$ GOOS=linux GOARCH=amd64 go build
```

将生成的二进制文件上传至服务器，然后启动：

```bash
$ ./my-xds
```



然后编写 Envoy 的配置文件，以从 xDS 服务端获取更多动态配置：

```yaml
admin:
  access_log_path: "/code/envoy.log"
  address:
    socket_address:
      address: 0.0.0.0
      port_value: 8081

dynamic_resources:
  cds_config:
    api_config_source:
      api_type: GRPC
      transport_api_version: v2
      grpc_services:
        envoy_grpc:
          cluster_name: xds_cluster
  lds_config:
    api_config_source:
      api_type: GRPC
      grpc_services:
      - envoy_grpc:
          cluster_name: xds_cluster
      set_node_on_first_message_only: true

node:
  cluster: service_bbc
  id: node1

static_resources:
  clusters:
  - name: xds_cluster
    connect_timeout: 1s
    hosts:
    - socket_address:
        address: 172.20.20.162
        port_value: 9002
    http2_protocol_options: {}
    type: STATIC
```

这里 CDS 和 LDS 都是从名为 `xds_cluster` 的静态 Cluster （地址是  `172.20.20.162:9002` ）中获取配置。

因为 Envoy 是运行在容器内的，我把 xDS 服务端运行在宿主机本机上了，所以这里要写宿主机的 IP 地址。当然也可以把 xDS 服务端通过 docker-compose 也跑在容器里，我这里为了方便，因为后面还要看 xDS 服务端的日志，直接在命令行里面看比较直观。

这里 node.id 是必须要指定的，上面的代码里有定义这个。



其他文件都不变。在另外一个命令行里启动容器：

```bash
$ sudo docker-compose up --build -d
```

启动后，可以看到 xDS 服务端的命令行中打印了几条日志，说明 Envoy 成功从 xDS 服务端获取到配置了（注：如果日志一直连续打印，说明出错了，出错原因需要查看 Docker 容器的日志）

测试访问：

```bash
$ curl http://localhost:8000/hello
world
```

我这里是完美的。

可以在 Envoy 的 admin 管理端查看最新的全部配置，我这里的地址是：http://fueltank-1:8081/config_dump 



---



## 卸载

```bash
$ sudo docker-compose down
```





---



## 总结

有一说一，现在网上关于 xDS 实现的文章还是比较少的，大多都是乱吹一通，然后就放一边不管了。并且官方文档也不是很直观，都是些数据格式，根本没有使用步骤，所以硬看肯定是不行的，我这篇也是查了很多资料，东拼西凑。

通过上面的动手操作，把 Envoy 的 API 使用套路弄懂了，流程也弄懂了，再去官方看那些数据格式就是小意思了。

完全理解 xDS 之后，可以自己包装一下 Envoy， 实现自己的代理，以替代 Nginx。另外，对 Istio 的理解也会进一步加深。

