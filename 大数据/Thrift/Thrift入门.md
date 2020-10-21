# Thrift 入门

Thrift 官方网站：https://thrift.apache.org/

Thrift 像 gRPC 一样，是一个 RPC 框架，支持多种语言，多用于大数据领域。



## 安装

先下载 Thrift 的源码包，下载地址：https://thrift.apache.org/download

我下载的是一个 tar.gz 包。

编译：

```bash
$ ./configure --with-boost=/usr/local
$ make
$ make check
$ sh test/test.sh
```

安装：

```bash
$ make install
```



## 测试

编辑 `shared.thrift`，内容如下：

```
namespace cl shared
namespace cpp shared
namespace d share // "shared" would collide with the eponymous D keyword.
namespace dart shared
namespace java shared
namespace perl shared
namespace php shared
namespace haxe shared
namespace netstd shared


struct SharedStruct {
  1: i32 key
  2: string value
}

service SharedService {
  SharedStruct getStruct(1: i32 key)
}
```



编辑 `tutorial.thrift`，内容如下：

```
include "shared.thrift"

namespace cl tutorial
namespace cpp tutorial
namespace d tutorial
namespace dart tutorial
namespace java tutorial
namespace php tutorial
namespace perl tutorial
namespace haxe tutorial
namespace netstd tutorial

typedef i32 MyInteger

const i32 INT32CONSTANT = 9853
const map<string,string> MAPCONSTANT = {'hello':'world', 'goodnight':'moon'}

enum Operation {
  ADD = 1,
  SUBTRACT = 2,
  MULTIPLY = 3,
  DIVIDE = 4
}

struct Work {
  1: i32 num1 = 0,
  2: i32 num2,
  3: Operation op,
  4: optional string comment,
}

exception InvalidOperation {
  1: i32 whatOp,
  2: string why
}

service Calculator extends shared.SharedService {

   void ping(),

   i32 add(1:i32 num1, 2:i32 num2),

   i32 calculate(1:i32 logid, 2:Work w) throws (1:InvalidOperation ouch),

   oneway void zip()

}
```

生成代码文件：

```
$ thrift -r --gen java tutorial.thrift
$ thrift -r --gen cpp tutorial.thrift
```







