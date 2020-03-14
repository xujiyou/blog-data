# Go 命令详解

官方教程：https://golang.org/cmd/go/

今天就开始翻译并学习这篇教程！！！！！！

go 是一个管理 go 语言代码的工具。

使用方式：

```
go <command> [arguments]
```

command 有：

- **bug** 反馈 bug
- **build** 构建项目
- **clean** 删除缓存和二进制文件
- **doc** 展示包内文档
- **env** 展示 go 的环境变量信息
- **fmt** 组织代码格式
- **generate** 通过处理源生成Go文件
- **get** 将依赖项添加到当前模块并安装它们
- **install** 编译安装依赖
- **list** 列出包或模块
- **mod** 模块管理
- **run** 编译运行
- **test** 测试
- **tool** 使用工具
- **version** 打印版本
- **vet** 报告包装中可能的错误

使用 "go help <command>" 来查看命令的更多信息

其他帮助主题：

- **buildmode** 构建模式
- **c** 在 C 和 GO 之间互相调用
- **cache** 构建和测试缓存
- **environment** 环境变量
- **filetype** 文件类型
- **go.mod** go.mod 文件
- **gopath** GOPATH环境变量
- **gopath-get** 旧的 GOPATH go get
- **goproxy** 模块代理协议
- **importpath** 导入路径语法
- **modules** modules, module versions, and more
- **module-get** module-aware go get
- **module-auth** module authentication using go.sum
- **module-private** module configuration for non-public modules
- **packages** package lists and patterns
- **testflag** testing flags
- **testfunc** testing functions



## 反馈 BUG

```bash
go bug
```

运行完成后就会用默认浏览器打开 Github Issue 页了。



## 编译包和依赖

使用方法：

```
go build [-o output] [-i] [build flags] [packages] 
```

builld 命令会编译包及其依赖，但不会把编译结果安装到 GOPATH/bin 目录中。

如果 build 命令后跟了多个 .go 文件，这种情况下，多个 go 文件仅会视作一个程序。

编译软件包时，build 会忽略以 '_test.go' 结尾的文件。

如果不指定 -o 参数，则输出的可执行文件的名称是以第一个源文件名进行命名的，比如执行 `go build ed.go rx.go` 将会产生名为 ed 或 ed.exe 的可执行文件。

当编译的包中没有 main 函数时，build 会编译软件包，但是会丢弃生成的可执行文件，仅用做检查是否可以构建软件包。

-o 参数可以是文件名，也可以是一个已经存在的目录，如果是一个文件名，就会在当前目录生成一个指定名称的可执行文件，如果是一个已经存在的目录（如果不存在，需要同时指定目录和文件名），就会将可执行文件输出到这个目录。

-i 选项将安装依赖包的编译产物（.a 文件）。

build flags 和 clean, get, install, list, run, 和 test 命令共享：

- **-a** 强制重建已经更新的软件包。
- **-n** 打印命令，但不运行它们。
- **-p n** 并行 build 的线程数量， 默认值为可用的CPU数量。
- **-race** 启用数据竞争检测（保证并发安全），仅支持linux/amd64, freebsd/amd64, darwin/amd64, windows/amd64，linux/ppc64le and linux/arm64 (only for 48-bit VMA).
- **-msan** 启用内存检查，仅支持linux/amd64, linux/arm64，在 Clang/LLVM 默认启用。
- **-v** 在编译时打印包名
- **-work** 打印临时工作目录的名称，在退出时也不要删除它。
- **-x** 打印命令
- **-asmflags '[pattern=]arg list'**  传递asm调用的参数
- **-buildmode mode** 构建模式，'go help buildmode' 查看更多信息
- **-compiler name** 编译器的名字，runtime.Compiler (gccgo or gc).
- **-gccgoflags '[pattern=]arg list'** gccgo 编译器的参数
- **-gcflags '[pattern=]arg list'** gc 编译器的参数
- **-installsuffix suffix** 在软件包安装目录名称中使用的后缀，为了使输出与默认版本分开。如果使用-race标志，则安装后缀将自动设置为race，或者，如果显式设置，则在其后附加_race。 对于-msan标志也是如此。 使用需要非默认编译标志的-buildmode选项，具有类似的效果。
- **-ldflags '[pattern=]arg list'** 用于传递每个go工具链接调用的参数。
- **-linkshared** 将与以前使用-buildmode = shared创建的共享库链接的构建代码。
- **-mod mode** 模块下载模式， readonly, vendor, or mod 。'go help modules' 查看更新信息
- **-modcacherw** 将新创建的目录保留在模块缓存中为可读写状态，而不是使其成为只读状态。
- **-modfile file** go.mod 和 go.sum
- **-pkgdir dir** 从自定义目录而不是通常的位置安装和加载所有软件包。
- **-tags tag,list** 以逗号分隔的构建标记列表，以在构建期间视为满意。 有关构建标记的更多信息，请参阅go / build软件包的文档中的构建约束说明。（Go的早期版本使用以空格分隔的列表，已弃用，但仍然可以识别。）
- **-trimpath** 
- **-toolexec 'cmd args'** 用于调用工具链程序（例如vet和asm）的程序。例如，不运行asm，而是运行go命令
  'cmd args /path/to/asm <arguments for asm>'。



