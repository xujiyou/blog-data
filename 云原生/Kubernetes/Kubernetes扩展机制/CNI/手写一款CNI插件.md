# 手写一款 CNI 插件

学会了使用 cnitool 之后，然后再来开发一款 CNI 插件。

这个插件模仿 ptp，实现创建一个 Veth 设备对，并实现相互之间能够通信。但是不搞那么花里胡哨的，直接点，简单粗暴。

插件名字简单点，叫 hello.

先搭个架子：

```go
package main

import (
   "github.com/containernetworking/cni/pkg/skel"
   "github.com/containernetworking/cni/pkg/version"
   bv "github.com/containernetworking/plugins/pkg/utils/buildversion"
)

func main() {
   skel.PluginMain(cmdAdd, cmdCheck, cmdDel, version.All, bv.BuildString("hello"))
}

func cmdAdd(args *skel.CmdArgs) error {
   return nil
}

func cmdCheck(args *skel.CmdArgs) error {
   return nil 
}

func cmdDel(args *skel.CmdArgs) error {
   return nil
}
```

