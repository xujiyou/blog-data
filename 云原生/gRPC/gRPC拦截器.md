# gRPC 拦截器

gRPC 的拦截器分流式拦截器和普通的拦截器。下面这两个例子实现了自定义的 token 验证

gRPC 中间件：https://github.com/grpc-ecosystem/go-grpc-middleware

下面先看普通的拦截器，主要使用到了 `UnaryInterceptor` 。

前情回顾： [Golang构建gRPC服务.md](Golang构建gRPC服务.md) 

服务端：

````go
package main

import (
	"context"
	"flag"
	"fmt"
	"google.golang.org/grpc"
	"google.golang.org/grpc/codes"
	"google.golang.org/grpc/metadata"
	"google.golang.org/grpc/status"
	"grpc-test/helloworld"
	"log"
	"net"
)

func main() {
	flag.Parse()
	lis, err := net.Listen("tcp", fmt.Sprintf(":%d", 9000))
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}

	grpcServer := grpc.NewServer(grpc.UnaryInterceptor(interceptor))
	helloworld.RegisterHelloServer(grpcServer, &XujiyouServer{})
	grpcServer.Serve(lis)
}

func interceptor(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo, handler grpc.UnaryHandler) (interface{}, error) {
	err := auth(ctx)
	if err != nil {
		return nil, err
	}
	// 继续处理请求
	return handler(ctx, req)
}

func auth(ctx context.Context) error {
	md, ok := metadata.FromIncomingContext(ctx)
	if !ok {
		return status.Errorf(codes.Unauthenticated, "无Token认证信息")
	}
	var (
		appid  string
		appkey string
	)
	if val, ok := md["appid"]; ok {
		appid = val[0]
	}
	if val, ok := md["appkey"]; ok {
		appkey = val[0]
	}
	if appid != "101010" || appkey != "i am key" {
		return status.Errorf(codes.Unauthenticated, "Token认证信息无效: appid=%s, appkey=%s", appid, appkey)
	}
	return nil
}
````

客户端：

```go
package main

import (
	"context"
	"google.golang.org/grpc"
	"grpc-test/helloworld"
	"log"
	"time"
)

type customCredential struct{}

func (c customCredential) RequireTransportSecurity() bool {
	return false
}

func main() {
	conn, _ := grpc.Dial(
		"0.0.0.0:9000",
		grpc.WithInsecure(),
		grpc.WithPerRPCCredentials(new(customCredential)),
		grpc.WithUnaryInterceptor(interceptor))

	defer conn.Close()

	client := helloworld.NewHelloClient(conn)

	resp, _ := client.SayName(context.TODO(), &helloworld.NameRequest{Query: "haha"})
	log.Println(resp)
}

func (c customCredential) GetRequestMetadata(ctx context.Context, uri ...string) (map[string]string, error) {
	return map[string]string{
		"appid":  "101010",
		"appkey": "i am key",
	}, nil
}

func interceptor(ctx context.Context, method string, req, reply interface{}, cc *grpc.ClientConn, invoker grpc.UnaryInvoker, opts ...grpc.CallOption) error {
	start := time.Now()
	err := invoker(ctx, method, req, reply, cc, opts...)
	log.Printf("method=%s req=%v rep=%v duration=%s error=%v\n", method, req, reply, time.Since(start), err)
	return err
}
```



流式拦截器：

前情回顾： [gRPC流式传输.md](gRPC流式传输.md) 

下面使用双向流来验证，主要使用 StreamInterceptor ：

服务端：

```go
package main

import (
	"context"
	"flag"
	"fmt"
	"google.golang.org/grpc"
	"google.golang.org/grpc/codes"
	"google.golang.org/grpc/metadata"
	"google.golang.org/grpc/status"
	"grpc-test/helloworld"
	"log"
	"net"
)

func main() {
	flag.Parse()
	lis, err := net.Listen("tcp", fmt.Sprintf(":%d", 9000))
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}

	grpcServer := grpc.NewServer(grpc.StreamInterceptor(streamAuth))
	helloworld.RegisterHelloServer(grpcServer, &XujiyouServer{})
	grpcServer.Serve(lis)
}

// StreamAuth .
func streamAuth (srv interface{}, ss grpc.ServerStream, info *grpc.StreamServerInfo, handler grpc.StreamHandler) error {
	log.Println("拦截成功")
	err := auth(ss.Context())
	if err != nil {
		return err
	}
	return handler(srv, ss)
}

func auth(ctx context.Context) error {
	md, ok := metadata.FromIncomingContext(ctx)
	if !ok {
		return status.Errorf(codes.Unauthenticated, "无Token认证信息")
	}
	var (
		appid  string
		appkey string
	)
	if val, ok := md["appid"]; ok {
		appid = val[0]
	}
	if val, ok := md["appkey"]; ok {
		appkey = val[0]
	}
	if appid != "101010" || appkey != "i am key" {
		return status.Errorf(codes.Unauthenticated, "Token认证信息无效: appid=%s, appkey=%s", appid, appkey)
	}
	return nil
}
```

客户端：

```go
package main

import (
	"context"
	"fmt"
	"google.golang.org/grpc"
	"grpc-test/helloworld"
	"log"
	"strconv"
	"time"
)

func main() {
	conn, err := grpc.Dial("localhost:9000", grpc.WithInsecure(), grpc.WithStreamInterceptor(streamInterceptor))
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

type customCredential struct{}

func (c customCredential) RequireTransportSecurity() bool {
	return false
}

func (c customCredential) GetRequestMetadata(ctx context.Context, uri ...string) (map[string]string, error) {
	return map[string]string{
		"appid":  "101010",
		"appkey": "i am key",
	}, nil
}

func streamInterceptor(ctx context.Context, desc *grpc.StreamDesc, cc *grpc.ClientConn, method string, streamer grpc.Streamer, opts ...grpc.CallOption) (grpc.ClientStream, error) {
	var credsConfigured bool
	for _, o := range opts {
		_, ok := o.(*grpc.PerRPCCredsCallOption)
		if ok {
			credsConfigured = true
		}
	}
	if !credsConfigured {
		log.Println("hee")
		opts = append(opts, grpc.PerRPCCredentials(customCredential{}))
	}
	s, err := streamer(ctx, desc, cc, method, opts...)
	if err != nil {
		return nil, err
	}
	return s, nil
}
```



