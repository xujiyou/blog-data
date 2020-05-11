# SDS

写服务端的代码，main.go ：

```go
package main

import (
	"fmt"
	sd "github.com/envoyproxy/go-control-plane/envoy/service/discovery/v2"
	"google.golang.org/grpc"

	"log"
	"net"
)

func main() {
	lis, err := net.Listen("tcp", fmt.Sprintf(":%d", 9000))
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}

	grpcServer := grpc.NewServer()
	sd.RegisterSecretDiscoveryServiceServer(grpcServer, &MySDS{})
	grpcServer.Serve(lis)
}
```

server.go：

```yaml
package main

import (
	"context"
	v2 "github.com/envoyproxy/go-control-plane/envoy/api/v2"
	auth "github.com/envoyproxy/go-control-plane/envoy/api/v2/auth"
	core "github.com/envoyproxy/go-control-plane/envoy/api/v2/core"
	sd "github.com/envoyproxy/go-control-plane/envoy/service/discovery/v2"
	"github.com/envoyproxy/go-control-plane/pkg/cache/types"
	"github.com/golang/protobuf/proto"
	"github.com/golang/protobuf/ptypes/any"
	"io"
	"io/ioutil"
	"log"
)

type MySDS struct {}

var typeUrl = "type.googleapis.com/envoy.api.v2.auth.Secret"

func (s *MySDS) DeltaSecrets(server sd.SecretDiscoveryService_DeltaSecretsServer) error {
	return nil
}

func (s *MySDS) StreamSecrets(server sd.SecretDiscoveryService_StreamSecretsServer) error {
	for {
		_, err := server.Recv()
		if err == io.EOF {
			return nil
		}
		if err != nil {
			log.Printf("failed to recv: %v", err)
			return err
		}
		//log.Printf("server recv: %s", in)
		resp := getResp()
		_ = server.Send(&resp)
	}
}

func (s *MySDS) FetchSecrets(ctx context.Context, request *v2.DiscoveryRequest) (*v2.DiscoveryResponse, error) {
	log.Println(request)
	resp := getResp()
	return &resp, nil
}

func getResp() v2.DiscoveryResponse {
	buff := proto.NewBuffer(nil)
	buff.SetDeterministic(true)

	fileByte, _ := ioutil.ReadFile("/home/admin/k8s-cluster/envoy/ssl/cert/server.crt")
	log.Println(string(fileByte))

	resources := []types.Resource{
		&auth.Secret{
			Name: "my_secret",
			Type: &auth.Secret_TlsCertificate{
				TlsCertificate: &auth.TlsCertificate{
					CertificateChain: &core.DataSource{
						Specifier: &core.DataSource_InlineString{
							InlineString: string(fileByte),
						},
					},
					PrivateKey: &core.DataSource{
						Specifier: &core.DataSource_Filename{
							Filename: "/home/admin/k8s-cluster/envoy/ssl/cert/server.key",
						},
					},
				},
			},
		},
	}


	err := buff.Marshal(resources[0])

	if err != nil {
		log.Fatal(err)
	}

	return v2.DiscoveryResponse{
		VersionInfo: "1.0",
		Resources: []*any.Any{
			{
				TypeUrl: typeUrl,
				Value: buff.Bytes(),
			},
		},
		TypeUrl: typeUrl,
	}
}
```

注意这里的 `DataSource` 有三种形式 ，file、string、byte。string 和 byte 都是通过读取文件得来的。

Envoy 配置如下，注意 `sds_config` ：

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
  secrets:
#    - name: server_cert
#      tls_certificate:
#        certificate_chain:
#          filename: /home/admin/k8s-cluster/envoy/ssl/cert/server.crt
#        private_key:
#          filename: /home/admin/k8s-cluster/envoy/ssl/cert/server.key
    - name: validation_context
      validation_context:
        trusted_ca:
          filename: /home/admin/k8s-cluster/envoy/ssl/cert/ca.crt
        trust_chain_verification: ACCEPT_UNTRUSTED
  listeners:
  - address:
      socket_address:
        address: 0.0.0.0
        port_value: 82
    listener_filters:
      - name: "envoy.filters.listener.tls_inspector"
        typed_config: {}
    filter_chains:
      - filter_chain_match:
          server_names: ["fueltank-1"]
        transport_socket:
          name: envoy.transport_sockets.tls
          typed_config:
            "@type": type.googleapis.com/envoy.api.v2.auth.DownstreamTlsContext
            require_client_certificate: true
            common_tls_context:
              tls_certificate_sds_secret_configs:
                name: my_secret
                sds_config:
                  api_config_source:
                    api_type: GRPC
                    grpc_services:
                      envoy_grpc:
                        cluster_name: sds_server
              validation_context_sds_secret_config:
                name: validation_context
        filters:
        - name: envoy.filters.network.client_ssl_auth
          typed_config:
            "@type": type.googleapis.com/envoy.config.filter.network.client_ssl_auth.v2.ClientSSLAuth
            auth_api_cluster: cert-service
            stat_prefix: cert
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
                    prefix: "/"
                  route:
                    cluster: ping-service
            http_filters:
              - name: envoy.filters.http.router
  clusters:
  - name: cert-service
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    load_assignment:
      cluster_name: cert-service
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: 127.0.0.1
                port_value: 6060
  - name: ping-service
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    load_assignment:
      cluster_name: ping-service
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: 127.0.0.1
                port_value: 9090
    transport_socket:
      name: envoy.transport_sockets.tls
      typed_config: {}
  - name: sds_server
    type: LOGICAL_DNS
    connect_timeout: 1s
    load_assignment:
      cluster_name: sds_server
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: 127.0.0.1
                port_value: 9000
    http2_protocol_options: {}
```

