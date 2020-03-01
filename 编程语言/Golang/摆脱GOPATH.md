# 摆脱GOPATH

GOPATH 这东西坑的一批，究其原因，主要是 golang 的模块化并不是和语言本身伴生的。于是社区开发了各种各样的包管理工具，要是挑起来，真是眼花缭乱。

但是时至今日，前景也很明朗了，官方推出了 vgo，我也不全看那些乱七八糟的包管理工具了。

还有，GOPATH 我再也不想设置了。



先干掉 GOPATH

用 Goland 创建项目，选 Go Modules(vgo)，选好项目目录，注意，这里随便选，随便选，不必选到 $GOPATH/src 低下。然后点 create。

项目创建完成后，在项目低下创建 main 目录，然后随便写个文件，这个文件里面就可以写 main 方法了。

也可以直接在项目跟目录下创建一个 go 文件，然后写 main 方法，不过这个文件的 package 一定要是 main。



##  Go Modules(vgo)

Modules是Go 1.11中新增的实验性功能，基于vgo演变而来，是一个新型的包管理工具。

以下命令会生成一个 `go.mod` 的文件：

```bash
$ go mod init <project-name>
```

创建 main.go :

```go
package main

import (
	"fmt"
	log "github.com/sirupsen/logrus"
)

func main() {
	fmt.Print("hello world")
	log.Debug("hah")
}
```

然后执行：

```bash
$ go build
```

这时候会自动下载依赖，并进行编译。

完成后，查看 go.mod 有什么变化：

```
module second

go 1.14

require github.com/sirupsen/logrus v1.4.2
```

发现依赖在 go.mod 中注明了。同时多了一个`go.sum`的文件

```
github.com/davecgh/go-spew v1.1.1/go.mod h1:J7Y8YcW2NihsgmVo/mv3lAwl/skON4iLHjSsI+c5H38=
github.com/konsorten/go-windows-terminal-sequences v1.0.1/go.mod h1:T0+1ngSBFLxvqU3pZ+m/2kptfBszLMUkC4ZK/EgS/cQ=
github.com/pmezard/go-difflib v1.0.0/go.mod h1:iKH77koFhYxTK1pcRnkKkqfTogsbg7gZNVY4sRDYZ/4=
github.com/sirupsen/logrus v1.4.2 h1:SPIRibHv4MatM3XXNO2BJeFLZwZ2LvZgfQ5+UNI2im4=
github.com/sirupsen/logrus v1.4.2/go.mod h1:tLMulIdttU9McNUspp0xgXVQah82FyeX6MwdIuYE2rE=
github.com/stretchr/objx v0.1.1/go.mod h1:HFkY916IF+rwdDfMAkV7OtwuqBVzrE8GR6GFx+wExME=
github.com/stretchr/testify v1.2.2/go.mod h1:a8OnRcib4nhh0OaRAV+Yts87kKdq0PP7pXfy6kDkUVs=
golang.org/x/sys v0.0.0-20190422165155-953cdadca894 h1:Cz4ceDQGXuKRnVBDTS23GTn/pU5OE2C0WrNTOYK1Uuc=
golang.org/x/sys v0.0.0-20190422165155-953cdadca894/go.mod h1:h1NjWce9XRLGQEsW7wpKNCjG9DtNlClVuFLEZdDNbEs=
```

go.sum不是一个锁文件，是一个模块版本内容的校验值，用来验证当前缓存的模块。go.sum包含了直接依赖和间接依赖的包的信息，比go.mod要多一些。



### go.mod

有四种指令：module，require，exclude，replace。

- module：模块名称
- require：依赖包列表以及版本
- exclude：禁止依赖包列表（仅在当前模块为主模块时生效）
- replace：替换依赖包列表 （仅在当前模块为主模块时生效）

### go mod 相关命令

```bash
$ go mod tidy #拉取缺少的模块，移除不用的模块。
$ go mod download #下载依赖包
$ go mod graph #打印模块依赖图
$ go mod vendor #将依赖复制到vendor下
$ go mod verify #校验依赖
$ go mod why #解释为什么需要依赖
$ go list -m -json all #依赖详情
```



真香！！！













