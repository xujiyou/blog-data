# NUMA



对于多处理器架构来说，有两种使用内存的方式，一种是 UMA（一致性内存访问），另一种就是 NUMA。

在 NUMA 架构中，一个Node中包括一组CPU核心，这组CPU核心共享一块内存。

NUMA 使用 Distance 这个概念来描述 Local Access 和 Remote Access。

查看各 Node 之间呢的 Distance 情况：

```
$ sudo yum install numactl
$ sudo numactl --hardware
```

结果：

```
available: 2 nodes (0-1)
node 0 cpus: 0 1 2 3 4 5 12 13 14 15 16 17
node 0 size: 64155 MB
node 0 free: 5618 MB
node 1 cpus: 6 7 8 9 10 11 18 19 20 21 22 23
node 1 size: 64481 MB
node 1 free: 496 MB
node distances:
node   0   1 
  0:  10  21 
  1:  21  10
```

查看 NUMA 命中率：

```bash
$ numastat
```

结果：

```
                           node0           node1
numa_hit              4289830759      5478545137
numa_miss              218278607        17581335
numa_foreign            17581335       218278607
interleave_hit             23437           23715
local_node            4289808668      5478484867
other_node             218300698        17641605
```

numa_foreign 表示使用的另一Node申请的内存。numa_miss 是本 Node 不能使用的内存，在两个 Node 的机器上，本 Node 的 numa_foreign 就是另外一个 Node 的 numa_miss。

other_node过高意味着需要重新规划numa.

























