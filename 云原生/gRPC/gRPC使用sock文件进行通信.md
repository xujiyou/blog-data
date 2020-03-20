# gRPC 使用 sock 文件进行通信

gRPC 默认使用 TCP 通信。如果想仅仅在本机进行进程间通信，就没必要过一层网络接口了，只需使用 unix socket 就可以了。

基于Unix域套接字，速度可以比TCP套接字快得多。

下面来测试。

代码跟随上一节： [gRPC认证.md](gRPC认证.md) 

服务端：

```go
package main

import (
	"flag"
	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials"
	"grpc-test/helloworld"
	"log"
	"net"
)

func main() {
	flag.Parse()
	serverAddress, err := net.ResolveUnixAddr("unix", "/tmp/default.sock")
	listen, listenErr := net.ListenUnix("unix", serverAddress)
	if listenErr != nil {
		log.Fatalf("listenErr: %v", listenErr)
	}
	log.Printf("listen to /tmp/default.sock")
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}

	certAndKey, _ := credentials.NewServerTLSFromFile("./etcd/etcd.pem", "./etcd/etcd-key.pem")

	grpcServer := grpc.NewServer(grpc.Creds(certAndKey))
	helloworld.RegisterHelloServer(grpcServer, &XujiyouServer{})
	_ = grpcServer.Serve(listen)
}
```

代码会自动创建 sock 文件。

客户端：

```go
package main

import (
	"context"
	"fmt"
	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials"
	"grpc-test/helloworld"
	"log"
	"net"
	"strconv"
	"time"
)

func UnixConnect(context.Context, string) (net.Conn, error) {
	unixAddress, err := net.ResolveUnixAddr("unix", "/tmp/default.sock")
	conn, err := net.DialUnix("unix", nil, unixAddress)
	return conn, err
}
func main() {
	cert, err := credentials.NewClientTLSFromFile("./etcd/etcd.pem", "127.0.0.1")

	conn, err := grpc.Dial("/tmp/default.sock", grpc.WithTransportCredentials(cert), grpc.WithContextDialer(UnixConnect))
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

