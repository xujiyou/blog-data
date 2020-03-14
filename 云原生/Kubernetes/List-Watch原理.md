# List-Watch 原理

Kubernetes 集群中，到处都在使用 List-Watch 来实现消息的实时性。包括 kube-apiserver 与 etcd 之间，kube-controller-manager 、kube-scheduler、kubelet 与 kube-apiserver 之间，另外 kubectl 或浏览器访问 kube-apiserver 时也可以使用 List-Watch 机制，比如：

```
GET /api/v1/watch/pods
```

或：

```bash
$ kubectl get pods --watch
```

那 Kubernetes 中是如何实现这种机制的那？其实使用的是 HTTP Streaming，即在 HTTP header 中加入 **Transfer-Encoding： chunked** ！

不过上边的结论都是网上的传说，不知道是不是真的。下面就来测试一下，往 header 中加一个字段就能实现长链接。

看代码：

```go
package main

import (
	"fmt"
	"github.com/emicklei/go-restful/v3"
	"log"
	"net/http"
	"time"
)

func main() {
	webService := new(restful.WebService)

	webService.Route(webService.GET("/hello").To(hello))

	restful.Add(webService)
	log.Print("Start listening on localhost:8080")
	log.Fatal(http.ListenAndServe(":8080", nil))
}

func hello(req *restful.Request, resp *restful.Response) {
	resp.Header().Set("X-Content-Type-Options", "nosniff")
	_, _ = fmt.Fprintf(resp, "world1\n")
	resp.Flush()
	time.Sleep(1 * time.Second)
	_, _ = fmt.Fprintf(resp, "world2\n")
	resp.Flush()
	time.Sleep(1 * time.Second)
	_, _ = fmt.Fprintf(resp, "world3\n")
	resp.Flush()
	time.Sleep(1 * time.Second)
	_, _ = fmt.Fprintf(resp, "world4\n")
}
```

代码中并木有看到 `Transfer-Encoding： chunked` 哇！！！！

其实再使用 `resp.Flush()` 时会自动往响应 header 中加入 `Transfer-Encoding： chunked` 。

`resp.Header().Set("X-Content-Type-Options", "nosniff")` 这句的作用是让浏览器也能看到效果。

浏览器或 curl 访问： http://127.0.0.1:8080/hello

![image-20200314195628494](../../resource/image-20200314195628494.png)