# RDS

RDS，即 Route discovery service。

CDS、LDS 是存在于 `DynamicResources` 中，而 RDS 存在于 `HttpConnectionManager` 中。

实现代码如下：

```go
package main

import (
	v2 "github.com/envoyproxy/go-control-plane/envoy/api/v2"
	route "github.com/envoyproxy/go-control-plane/envoy/api/v2/route"
	"github.com/envoyproxy/go-control-plane/pkg/cache"
)

func BuildRouter() []cache.Resource  {

	return []cache.Resource{
		&v2.RouteConfiguration{
			Name: "my-route",
			VirtualHosts: []*route.VirtualHost{
				{
					Name: "my-virtual-host",
					Domains: []string{ "*" },
					Routes: []*route.Route{
						{
							Match: &route.RouteMatch{
								PathSpecifier: &route.RouteMatch_Prefix{
									Prefix: "/",
								},
							},
							Action: &route.Route_Route{
								Route: &route.RouteAction{
									ClusterSpecifier: &route.RouteAction_Cluster{
										Cluster: "service_bbc",
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



