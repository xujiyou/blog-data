# 动手理解 Docker 原理

首先 Docker 主要使用了 Linux 内核中的 Namespace 和 cgroup 功能。

linux 共有以下六种不同类型的 Namespaces：

| 类型              | 系统调用参数  | 内核版本 |
| ----------------- | ------------- | -------- |
| Mount Namespace   | CLONE_NEWNS   | 2.4.19   |
| UTS Namespace     | CLONE_NEWUTS  | 2.6.19   |
| IPC Namespace     | CLONE_NEWIPC  | 2.6.19   |
| PID Namespace     | CLONE_NEWIPC  | 2.6.19   |
| Network Namespace | CLONE_NEWNET  | 2.6.29   |
| User Namespace    | CLONE_NEWUSER | 3.8      |

Namespace 的 api 主要用到了以下三个系统调用：

- clone() 创建新进程，根据系统参数调用来判断哪些类型的 Namespace 会被创建，而且他们的子进程也会包含到这些 Namespace 中。
- unshare() 将进程移出 Namespce
- setns() 将进程加入到 Namespace

## UTS Namespace

UTS Namespace 主要用来隔离 nodename 和 dominname 两个系统标示。

写代码，在 macOS 上写代码请注意：https://here2say.com/36/

在 macOS 上创建 Go Mudeules项目：docker-uts ，然后写代码，main.go：

```go
// +build linux

package main

import (
	"log"
	"os"
	"os/exec"
	"syscall"
)

func main() {
	cmd := exec.Command("sh")
	cmd.SysProcAttr = &syscall.SysProcAttr{
		Cloneflags: syscall.CLONE_NEWUTS,
	}
	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr

	if err := cmd.Run(); err != nil {
		log.Fatal(err)
	}
}
```

然后使用命令编译：

```bash
$ GOOS=linux go build
```

编译好之后，将生成的二进制文件发到 Linux 系统上。

在 Linux 系统上，直接执行 `./docker-uts`

执行完成后，会进入一个 sh 命令行。

使用命令查看进程树和当前进程 ID：

```bash
$ pstree -pl
systemd(1)───sshd(2285)───sshd(11708)───sshd(11739)───bash(11747)───sudo(5184)───docker-uts(5186)─┬─sh(5192)
$ echo $$
5192
```

然后查看当前进程和父进程是否都在一个 UTS Namespace 中：

```bash
$ /proc/5192/ns/uts
uts:[4026532316]
$ readlink /proc/5186/ns/uts
uts:[4026531838]
```

发现不在同一个 UTS Namespace 中。

然后再在这个 sh 中修改 hostname：

```bash
$ hostname -b bird
$ hostname
bird
```

然后重新启动另一个命令行，查看 hostname:

```bash
$ hostname
fueltank-1.cloud.bbdops.com
```

可以看到外部的 hostnam 井没有被内部的修改所影响，由此可了解 UTS Namespac 的作用

