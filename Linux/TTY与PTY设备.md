---
title: TTY与PTY设备
date: 2020-07-18 14:12:41
tags:
---

在 Linux 的 /dev 目录下，除了创用的块设备外，还有很多 TTY 设备。

参考：https://segmentfault.com/a/1190000009082089

查看当前连接使用的是哪个 tty 设备：

```bash
$ tty
/dev/pts/0
```

每个连接使用的 tty 设备都不一样，可以通过打开多个窗口进行验证。

```bash
$ tty # 在另一个窗口执行
/dev/pts/1
```



往 tty 设备中直接写入和从命令行直接写入是一样的：

```bash
$ echo aaa > /dev/pts/1 # 在第一个窗口执行，第二个窗口会显示信息
```



## ssh 远程访问示意图

```
 +----------+       +------------+
 | Keyboard |------>|            |
 +----------+       |  Terminal  |
 | Monitor  |<------|            |
 +----------+       +------------+
                          |
                          |  ssh protocol
                          |
                          ↓
                    +------------+
                    |            |
                    | ssh server |--------------------------+
                    |            |           fork           |
                    +------------+                          |
                        |   ↑                               |
                        |   |                               |
                  write |   | read                          |
                        |   |                               |
                  +-----|---|-------------------+           |
                  |     |   |                   |           ↓
                  |     ↓   |      +-------+    |       +-------+
                  |   +--------+   | pts/0 |<---------->| shell |
                  |   |        |   +-------+    |       +-------+
                  |   |  ptmx  |<->| pts/1 |<---------->| shell |
                  |   |        |   +-------+    |       +-------+
                  |   +--------+   | pts/2 |<---------->| shell |
                  |                +-------+    |       +-------+
                  |    Kernel                   |
                  +-----------------------------+
```



## PTY 与 PTMX

pty（pseudo terminal device）由两部分构成，ptmx是master端，pts是slave端，

进程可以通过调用API请求ptmx创建一个pts，然后将会得到连接到ptmx的读写fd和一个新创建的pts，

ptmx在内部会维护该fd和pts的对应关系，随后往这个fd的读写会被ptmx转发到对应的pts。

下面的命令可以看到sshd已经打开了/dev/ptmx：

```bash
$ sudo lsof /dev/ptmx
```



## tmux 原理图

```
 +----------+       +------------+
 | Keyboard |------>|            |
 +----------+       |  Terminal  |
 | Monitor  |<------|            |
 +----------+       +------------+
                          |
                          |  ssh protocol
                          |
                          ↓
                    +------------+
                    |            |
                    | ssh server |--------------------------+
                    |            |           fork           |
                    +------------+                          |
                        |   ↑                               |
                        |   |                               |
                  write |   | read                          |
                        |   |                               |
                  +-----|---|-------------------+           |
                  |     ↓   |                   |           ↓
                  |   +--------+   +-------+    |       +-------+  fork   +-------------+
                  |   |  ptmx  |<->| pts/0 |<---------->| shell |-------->| tmux client |
                  |   +--------+   +-------+    |       +-------+         +-------------+
                  |   |        |                |                               ↑
                  |   +--------+   +-------+    |       +-------+               |
                  |   |  ptmx  |<->| pts/2 |<---------->| shell |               |
                  |   +--------+   +-------+    |       +-------+               |
                  |     ↑   |  Kernel           |           ↑                   |
                  +-----|---|-------------------+           |                   |
                        |   |                               |                   |
                        |w/r|   +---------------------------+                   |
                        |   |   |            fork                               |
                        |   ↓   |                                               |
                    +-------------+                                             |
                    |             |                                             |
                    | tmux server |<--------------------------------------------+
                    |             |
                    +-------------+
```



## TTY和PTS的区别

从上面的流程中应该可以看出来了，对用户空间的程序来说，他们没有区别，都是一样的；从内核里面来看，pts的另一端连接的是ptmx，而tty的另一端连接的是内核的终端模拟器，ptmx和终端模拟器都只是负责维护会话和转发数据包；再看看ptmx和内核终端模拟器的另一端，ptmx的另一端连接的是用户空间的应用程序，如sshd、tmux等，而内核终端模拟器的另一端连接的是具体的硬件，如键盘和显示器。



## 常见的TTY配置

先先来看看当前tty的所有配置：

```
$ stty -a
speed 38400 baud; rows 66; columns 191; line = 0;
intr = ^C; quit = ^\; erase = ^?; kill = ^U; eof = ^D; eol = <undef>; eol2 = <undef>; swtch = <undef>; start = ^Q; stop = ^S; susp = ^Z; rprnt = ^R; werase = ^W; lnext = ^V; flush = ^O;
min = 1; time = 0;
-parenb -parodd -cmspar cs8 -hupcl -cstopb cread -clocal -crtscts
-ignbrk -brkint -ignpar -parmrk -inpck -istrip -inlcr -igncr icrnl ixon -ixoff -iuclc -ixany -imaxbel -iutf8
opost -olcuc -ocrnl onlcr -onocr -onlret -ofill -ofdel nl0 cr0 tab0 bs0 vt0 ff0
isig icanon iexten echo echoe echok -echonl -noflsh -xcase -tostop -echoprt echoctl echoke
```



## 最大连接数

查看最大连接数：

```bash
$ cat /proc/sys/kernel/pty/max
4096
```

这是一个内核参数，可以通过修改 kernel.pty.max 内核参数修改改值。





