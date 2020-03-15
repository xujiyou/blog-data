# go-restful 入门

Kube-apiserver 就是用的这个框架，所以来学学。

先上两个例子：

1. hello world

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
// GET http://localhost:8080/hello

func main() {
	ws := new(restful.WebService)
	ws.Route(ws.GET("/hello").To(hello))
	restful.Add(ws)
	log.Fatal(http.ListenAndServe(":8080", nil))
}

func hello(req *restful.Request, resp *restful.Response) {
	io.WriteString(resp, "world")
}
```

2. 获取 url 中的参数：

```go
package main

import (
	"fmt"
	restful "github.com/emicklei/go-restful"
	"io"
	"log"
	"net/http"
)

type UserResource struct {
	id string
	name string
}

type User struct {
	name string
}

func main() {
	var u = UserResource{id: "123", name: "xujiyou"}

	ws := new(restful.WebService)
	ws.
		Path("/users").
		Consumes(restful.MIME_XML, restful.MIME_JSON).
		Produces(restful.MIME_JSON, restful.MIME_XML)

	ws.Route(ws.GET("/{user-id}").To(u.findUser).
		Doc("get a user").
		Param(ws.PathParameter("user-id", "identifier of the user").DataType("string")).
		Writes(User{name: "xujiyou"}))

	restful.Add(ws)
	log.Fatal(http.ListenAndServe(":8080", nil))

}

func (u UserResource) findUser(request *restful.Request, response *restful.Response) {
	id := request.PathParameter("user-id")
	fmt.Println(id)
	io.WriteString(response, id)
}
```

3. 返回 Json：

```go
package server

import (
	"encoding/json"
	"github.com/emicklei/go-restful/v3"
	"io"
	"log"
	"net/http"
)

type Book struct {
	Name string `json:"name"`
	Id string `json:"id"`
}

func Run(subCommand SubCommand) {
	webService := new(restful.WebService)

	switch subCommand {
	case FILE:
		webService.Route(webService.GET("/object").To(hello))
	case MONGODB:
		webService.Route(webService.GET("/object").To(mongo))
	}

	restful.Add(webService)
	log.Print("Start listening on localhost:8080")
	log.Fatal(http.ListenAndServe(":8080", nil))
}

func hello(req *restful.Request, resp *restful.Response) {
	book := Book{Name: "k8s", Id: "1111"}
	_ = json.NewEncoder(resp).Encode(book)
}

func mongo(req *restful.Request, resp *restful.Response) {
	_, _ = io.WriteString(resp, "mongo return")
}
```

