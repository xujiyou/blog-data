# gRPC 认证

gRPC 有两种认证方式 **SSL/TLS** 、**Token**

## SSL/TLS

证书生成过程不说了，使用 openssl 很简单，这里我用之前的 etcd 证书做测试。

前提代码： [gRPC流式传输.md](gRPC流式传输.md) 

服务端启动代码：

```go
package main

import (
	"flag"
	"fmt"
	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials"
	"grpc-test/helloworld"
	"log"
	"net"
)

func main() {
	flag.Parse()
	listen, err := net.Listen("tcp", fmt.Sprintf(":%d", 9000))
	log.Printf("listen of 9000")
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}

	certAndKey, _ := credentials.NewServerTLSFromFile("./etcd/etcd.pem", "./etcd/etcd-key.pem")

	grpcServer := grpc.NewServer(grpc.Creds(certAndKey))
	helloworld.RegisterHelloServer(grpcServer, &XujiyouServer{})
	_ = grpcServer.Serve(listen)
}

```

客户端代码：

```go
package main

import (
	"context"
	"fmt"
	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials"
	"grpc-test/helloworld"
	"log"
	"strconv"
	"time"
)

func main() {
	cert, err := credentials.NewClientTLSFromFile("./etcd/etcd.pem", "127.0.0.1")

	conn, err := grpc.Dial("localhost:9000", grpc.WithTransportCredentials(cert))
	if err != nil {
		_ = fmt.Errorf("did not connect: %v", err)
	}
	defer conn.Close()

	client := helloworld.NewHelloClient(conn)

	stream, clientErr := client.SayName(context.Background())
	if clientErr != nil {
		_ = fmt.Errorf("did not stream: %v", clientErr)
	}

	for i := 0; i < 10; i++ {
		err := stream.Send(&helloworld.NameRequest{Query: "data" + strconv.Itoa(i), ResultPerPage: 1, PageNumber: 2})
		if err != nil {
			_ = fmt.Errorf("%v", err)
		}
		time.Sleep(300 * time.Millisecond)
		reply, _ := stream.Recv()
		log.Printf("client recv: %s", reply)
	}
	_ = stream.CloseSend()
}

```

注意 NewClientTLSFromFile 中的 ServerName 要填证书中允许的 DNS 或 IP。



## Token 认证

token 认证参考： [gRPC拦截器.md](gRPC拦截器.md) 