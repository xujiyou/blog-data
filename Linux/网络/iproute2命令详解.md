# iproute2 命令详解

官方仓库：https://github.com/shemminger/iproute2

CentOS安装：

```bash
$ sudo yum install -y iproute
```

Wiki:https://wiki.linuxfoundation.org/networking/iproute2





## IP 命令使用方法

IP 命令格式：

```
Usage: ip [ OPTIONS ] OBJECT { COMMAND | help }
       ip [ -force ] -batch filename
where  OBJECT := { link | address | addrlabel | route | rule | neigh | ntable |
                   tunnel | tuntap | maddress | mroute | mrule | monitor | xfrm |
                   netns | l2tp | fou | macsec | tcp_metrics | token | netconf | ila |
                   vrf | sr | nexthop }
       OPTIONS := { -V[ersion] | -s[tatistics] | -d[etails] | -r[esolve] |
                    -h[uman-readable] | -iec | -j[son] | -p[retty] |
                    -f[amily] { inet | inet6 | mpls | bridge | link } |
                    -4 | -6 | -I | -D | -M | -B | -0 |
                    -l[oops] { maximum-addr-flush-attempts } | -br[ief] |
                    -o[neline] | -t[imestamp] | -ts[hort] | -b[atch] [filename] |
                    -rc[vbuf] [size] | -n[etns] name | -N[umeric] | -a[ll] |
                    -c[olor]}
```

OBJECT 中，有几个是熟悉的，但大部分不熟悉。



#### OPTIONS

OPTIONS 影响ip的一般行为和输出。所有选项都以“-”字符开头，可以长格式和缩写形式使用。目前有以下选项可用：

1. -V, -Version 打印版本

2. -s, -stats, -statistics 输出更多详细信息

   > 可以重复此选项来获取更详细的信息，比如 `ip -s -s address`

3. -f, -family {inet, inet6, link} 强制使用哪个协议

   如果不存在此选项，将从其他命令行参数猜测要使用的协议族输出。如果某个协议族中的某个协议的其余部分不能提供足够的信息，则该协议或协议族的其他成员不能提供足够的信息。Link是一个特殊的族标识符，意味着不涉及网络协议。此选项有几个快捷方式，如下所示：

   -4 --- -family inet 的简称.

   -6 --- -family inet6 的简称.

   -0 --- -family link 的简称.

   -o, -oneline --- 通过将任何换行符替换为“\”字符，将输出记录格式化为单行。

4. -r, -resolve 使用系统名称解析输出DNS名称



#### OBJECT

- link --- 物理或逻辑网络设备

- address --- 设备上的IPv4或IPv6地址

- neighbour --- ARP或NDISC缓存项

- route --- 路由表项

- rule --- 数据库中的路由规则（rule in routing policy database.）

- maddress --- 多播地址。

- mroute --- 多播路由缓存项。

- tunnel --- IP隧道

> 所有对象的名称可以用全称或缩写形式书写。IE:address可以缩写为addr或只是a。但是如果你在脚本中使用这些命令，你应该养成一个习惯，那就是始终使用操作的完整规范。使用缩写可以使命令行易于使用，但很难理解脚本中的逻辑。既然你可能不是唯一一个需要处理脚本的人，那么你应该努力使它们尽可能完整。



## ip link

ip link 只有两个命令：set 和 show。





































