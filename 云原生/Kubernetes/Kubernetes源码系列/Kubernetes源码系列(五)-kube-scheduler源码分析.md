# Kubernetes 源码系列(五) --- kube-scheduler 源码分析

kube-scheduler 是相对独立的一个组件，主动访问 kube-apiserver，寻找等待调度的 pod，然后通过一系列调度算法寻找哪个 node 适合跑这个 pod，然后将这个 pod 和 node 的绑定关系发给 kube-apiserver，从而完成了调度的过程。

从高level看，scheduler的源码可以分为3层：

- `cmd/kube-scheduler/scheduler.go`: main() 函数入口位置，在scheduler过程开始被调用前的一系列初始化工作。
- `pkg/scheduler/scheduler.go`: 调度框架的整体逻辑，在具体的调度算法之上的框架性的代码。
- `pkg/scheduler/core/generic_scheduler.go`: 具体的计算哪些node适合跑哪些pod的算法。

先来看下 main.go ：

```go
package main

import (
	"math/rand"
	"os"
	"time"

	"github.com/spf13/pflag"

	cliflag "k8s.io/component-base/cli/flag"
	"k8s.io/component-base/logs"
	_ "k8s.io/component-base/metrics/prometheus/clientgo"
	"k8s.io/kubernetes/cmd/kube-scheduler/app"
)

func main() {
	rand.Seed(time.Now().UnixNano())

	command := app.NewSchedulerCommand()

	pflag.CommandLine.SetNormalizeFunc(cliflag.WordSepNormalizeFunc)
	
	logs.InitLogs()
	defer logs.FlushLogs()

	if err := command.Execute(); err != nil {
		os.Exit(1)
	}
}
```

`rand.Seed(time.Now().UnixNano())` 这句保证生成的随机数一定是随机的。

然后初始化配置选项和日志。最后执行靠的是 `command := app.NewSchedulerCommand()` 和 `command.Execute()`