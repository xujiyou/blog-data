# cnitool 使用方式

官方文档地址：https://github.com/containernetworking/cni/tree/master/cnitool

不过这个文档有点错误，直接执行命令的话通不过。

下面是我的实际操作，实测有效。

---



安装 cnitool 二进制命令：

```
$ go get github.com/containernetworking/cni
$ go install github.com/containernetworking/cni/cnitool
```

然后把 `$GOPATH/go/bin` 中的 `cnitool` 拷贝到 `/usr/bin`



编译官方提供的全部 cni 插件：

```
$ git clone https://github.com/containernetworking/plugins.git
$ cd plugins
$ ./build_linux.sh
```

执行完成后，会在当前目录生成一个 `bin` 目录，里面存有好多二进制的 cni 插件。



然后编写配置文件(这里官方文档是错误的)：

```
$ cat /etc/cni/net.d/10-myptp.conflist
{
    "cniVersion": "0.4.0",
    "name": "myptp",
    "plugins": [
        {
            "type": "ptp",
            "ipMasq": true,
            "ipam": {
                "type": "host-local",
                "subnet": "172.16.29.0/24",
                "routes": [
                    {
                        "dst": "0.0.0.0/0"
                    }
                ]
             }
        }
    ]
}
```

ptp 插件用于创建 veth 设备对，关于 veth 设备对可以看： [一次网络命令实践.md](../../../../Linux/Shell/一次网络命令实践.md) 



创建一个网络命名空间：

```bash
$ sudo ip netns add testing
```



在刚刚创建的命名空间内执行插件：

```bash
$ sudo CNI_PATH=./bin cnitool add myptp /var/run/netns/testing
```



检查效果 (ONLY for spec v0.4.0+)(亲测无效)：

```bash
$ sudo CNI_PATH=./bin cnitool check myptp /var/run/netns/testing
```



测试：

```bash
$ sudo ip -n testing addr
4: eth0@if528: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 22:77:81:25:4e:54 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.16.29.2/24 brd 172.16.29.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::2077:81ff:fe25:4e54/64 scope link 
       valid_lft forever preferred_lft forever
```

这条命令会列出网络命名空间 testing 内的网卡，我这里看到了一个名为 eth0@if528 的网卡

然后查看本机原始命名空间内的网卡：

```bash
$ ip a
528: vethbb097e6c@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 32:b3:c1:20:2d:b3 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.16.29.1/32 brd 172.16.29.1 scope global vethbb097e6c
       valid_lft forever preferred_lft forever
```

可以看到两个命名空间内的两个网卡遥相呼应。



测试 testing 命名空间内的网卡是否可以工作：

```bash
$ sudo ip netns exec testing ping -c 4 www.baidu.com
```

实测是可以工作的，ping 的 -c 参数指定了发包次数。



清理：

```bash
$ sudo CNI_PATH=./bin cnitool del myptp /var/run/netns/testing
$ sudo ip netns del testing
```



---



上面已经测试了 ptp 插件。下面开始想办法开发一个自己的插件，并运行起来。

上面在官方的插件代码里，有一个示例插件，在 `plugins/sample` 包下，下面先编译这个包，让后将其放入 bin 目录(先不该动任何代码)：

```bash
$ cd plugins/sample/
$ go build 
$ cd ../../
$ cp plugins/sample/sample ./bin
```

然后编写配置文件，在 `plugins/sample/sample_linux_test.go` 这个文件中有配置，需要对其进行改造：

```bash
$ cat /etc/cni/net.d/10-sample.conflist
{
    "cniVersion": "0.4.0",
    "name": "sample",
    "plugins": [
        {
            "type": "sample",
            "anotherAwesomeArg": "haha",
            "prevResult": {
                        "interfaces": [
                                 {
                                          "name": "eth0",
                                          "sandbox": "/var/run/netns/test"
                                 }
                         ],
                         "ips": [
                                 {
                                         "version": "4",
                                         "address": "10.0.0.2/24",
                                         "gateway": "10.0.0.1",
                                         "interface": 0
                                 }
                         ],
                         "routes": []
                 }
        }
    ]
}
```

然后使用 cnitool 来执行 sample 插件：

```bash
$ sudo ip netns add test
$ sudo CNI_PATH=./bin cnitool add sample /var/run/netns/test
```

测试：

```bash
$ sudo ip -n test addr
```

OK，这样 就可以自定义 CNI 插件了。



---



## cnitool 源码分析

cnitool 的源码在 https://github.com/containernetworking/cni/blob/master/cnitool/cnitool.go

一共就100多行，很简单。

其中，最关键的几句话：

```go
  cninet := libcni.NewCNIConfig(filepath.SplitList(os.Getenv(EnvCNIPath)), nil)

	rt := &libcni.RuntimeConf{
		ContainerID:    containerID,
		NetNS:          netns,
		IfName:         ifName,
		Args:           cniArgs,
		CapabilityArgs: capabilityArgs,
	}

	switch os.Args[1] {
	case CmdAdd:
		result, err := cninet.AddNetworkList(context.TODO(), netconf, rt)
		if result != nil {
			_ = result.Print()
		}
		exit(err)
	case CmdCheck:
		err := cninet.CheckNetworkList(context.TODO(), netconf, rt)
		exit(err)
	case CmdDel:
		exit(cninet.DelNetworkList(context.TODO(), netconf, rt))
	}
```

可以看出，其实就是传入配置，然后调用的插件的三个方法。。。

