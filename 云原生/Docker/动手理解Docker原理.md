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

在 Linux 系统上，直接执行 :

```bash
$ sudo ./docker-uts
```



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



## PID Namespace

PID Namespace 是用来隔离进程的，同样一个进程在不同的 Namespace 里面拥有不同的 PID。

修改刚才的代码，加上 syscall.CLONE_NEWPID ：

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
		Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWPID,
	}
	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr

	if err := cmd.Run(); err != nil {
		log.Fatal(err)
	}
}
```

重新打包，上传。然后重新执行：

```
$ sudo ./docker-uts
```

然后：

```
$ echo $$
1
```

这样子就会 发现当前进程ID变成 1 了。



## Mount Namespace

Mount Namespace 用来隔离各个进程看到的挂载点视图。在不同 Namespace 的进程中，看到的文件系统层次是不 样的。在 Mount Namespace 调用 mount（）和 umount（） 仅仅只会影响当前 Namespace 内的文件系统，而对全局的文件系统是没有影响的。

对上面的代码加入 syscall.CLONE_NEWNS

重新打包，上传。然后运行。进入新命令行后，依次运行：

```bash
$ ls /proc
$ mount -t proc proc /proc
$ ls /proc
$ ps -ef
```

观察结果

Docker Volume 是利用了这个特性。



## User Namespace

User Namespace 主要隔离的是用户组 ID，也就是说，一个进程的 User ID 和 Grroup ID 在 User Namespace 内外是 不同的 。

将上面的代码加入 syscall.CLONE_NEWUSER 。

重新打包，上传。然后重新执行：

```bash
$ sudo id
$ sudo ./docker-uts
```

然后在容器内：

```bash
$ id
```

可以看到，前后的 id 是不同的。



## Network Namespace

Network Namespace 不止用来隔离网络设备，还可以用来隔离IP地址端口！！！

关于网络命名空间可以看我的另一篇文章： [一次网络命令实践.md](../../Linux/Shell/一次网络命令实践.md) 



## Cgroups

上面理解了进程是如何隔离出单独的空间的，但如何限制空间的大小哪？这就要用到 Linux 的 Cgroups 技术了。

Linux Cgroups 提供了对一组进程及将来的子进程的资源限制、控制和统计能力，这些资源包括CPU、内存、储存、网络等，通过 Cgroups，可以实时的监控进程的监控和统计信息。

Cgroups 的三个组件：

- 首先，Cgroups 肯定要包含一组进程，并把这组进程与Linux subsystem的各种配置关联起来。

- subsystem 是一组资源控制的模块，包含以下几项：

  - blkio 对块设备的输入输出的访问
  - cpu 限制 CPU 的使用
  - cpuacct 统计进程的 CPU 占用
  - cpuset 设置进程可以使用的 cpu 和 内存
  - devices 控制对设备的访问
  - freezer 挂起恢复进程
  - memory 控制内存的使用
  - net_cls 对进程的网络包分类，以便做限流和监控
  - ns 创建新的 Cgroups

  查看内核支持哪些 subsystem：

  ```bash
  $ sudo yum install -y libcgroup-pam libcgroup-tools
  $ lssubsys -a
  cpuset
  cpu,cpuacct
  blkio
  memory
  devices
  freezer
  net_cls,net_prio
  perf_event
  hugetlb
  pids
  ```

- hierarchy 的功能是把一组 cgroup 串成 个树状的结构，一个这样的树便是一个 hierarchy ，通过这种树状结构， Cgroups 可以做到继承 比如，系统对一组定时的任务进程通过 cgroupl 限制了 CPU 的使用率，然后其中有一个定时 dump 日志的进程还需要限制磁盘 IO ，为了避免限制了磁盘 IO 之后影响到其他进程，就可以创建 cgroup2 ，使其继承于 cgroupl 井限制磁盘的 IO ，这样 cgroup2 便继承了 cgroupl 中对 CPU 使用率的限制，并且增加了磁盘 IO 的限制而不影响到 cgroupl 中的其他进程。









