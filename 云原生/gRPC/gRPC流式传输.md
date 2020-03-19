# gRPC 流式传输

下面分服务端流，客户端流，双向流来学习。基础是  [Golang构建gRPC服务.md](Golang构建gRPC服务.md) 



## 服务端流

看下面 stream 关键字的位置：

```protobuf
syntax = "proto3";

option objc_class_prefix = "HLW";

package helloworld;

service Hello {
    rpc sayName(NameRequest) returns (stream NameReply) {};
}

message NameRequest {
    string query = 1;
    int32 page_number = 2;
    int32 result_per_page = 3;
}

message NameReply {
    string message = 1;
}
```

然后修改服务方法：

```go
package main

import (
	"fmt"
	"grpc-test/helloworld"
	"strconv"
	"time"
)

type XujiyouServer struct {}

func (s *XujiyouServer) SayName(nameRequest *helloworld.NameRequest, server helloworld.Hello_SayNameServer) error {
	for i := 0; i < 10; i++ {
		err := server.Send(&helloworld.NameReply{Message: "Hello " + nameRequest.Query + strconv.Itoa(i)})
		if err != nil {
			_ = fmt.Errorf("%s", err)
		}
		time.Sleep(1 * time.Second)
	}
	return nil
}

```



## 客户端流 

修改 proto 文件，注意 stream 关键字位置：

```protobuf
syntax = "proto3";

option objc_class_prefix = "HLW";

package helloworld;

service Hello {
    rpc sayName(stream NameRequest) returns (NameReply) {};
}

message NameRequest {
    string query = 1;
    int32 page_number = 2;
    int32 result_per_page = 3;
}

message NameReply {
    string message = 1;
}
```

修改服务端实现：

```go
import (
	"grpc-test/helloworld"
	"io"
	"log"
	"strings"
)

type XujiyouServer struct {}

func (s *XujiyouServer) SayName(server helloworld.Hello_SayNameServer) error {
	var names []string
	for {
		in, err := server.Recv()
		if err == io.EOF {
			_ = server.SendAndClose(&helloworld.NameReply{Message: "You send data is ' "+ strings.Join(names, ",") + "'"})
			return nil
		}
		if err != nil {
			log.Printf("failed to recv: %v", err)
			return err
		}
		names = append(names, in.Query)
	}
}
```

添加客户端：

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
	conn, err := grpc.Dial("localhost:9000", grpc.WithInsecure())
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
	}

	reply, closeRrr := stream.CloseAndRecv()
	if closeRrr != nil {
		fmt.Printf("failed to recv: %v", closeRrr)
		return
	}
	log.Printf("Greeting: %s", reply.Message)
}

```

先运行服务端，在运行客户端，查看效果。



## 双向流

修改 proto：

```protobuf
syntax = "proto3";

option objc_class_prefix = "HLW";

package helloworld;

service Hello {
    rpc sayName(stream NameRequest) returns (stream NameReply) {};
}

message NameRequest {
    string query = 1;
    int32 page_number = 2;
    int32 result_per_page = 3;
}

message NameReply {
    string message = 1;
}
```

修改服务端：

```go
import (
	"grpc-test/helloworld"
	"io"
	"log"
)

type XujiyouServer struct {}

func (s *XujiyouServer) SayName(server helloworld.Hello_SayNameServer) error {
	for {
		in, err := server.Recv()
		if err == io.EOF {
			return nil
		}
		if err != nil {
			log.Printf("failed to recv: %v", err)
			return err
		}
		log.Printf("server recv: %s", in.Query)
		_ = server.Send(&helloworld.NameReply{Message: "You send data is ' "+ in.Query + "'"})
	}
}
```

修改客户端：

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
	conn, err := grpc.Dial("localhost:9000", grpc.WithInsecure())
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



OK，都可以运行了，基于此，可以实现很多有意思的设计。



