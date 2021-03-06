# xfs 文件系统详解

引用维基百科对文件系统的定义：“计算机的文件系统是一种存储和组织计算机数据的方法，它使得对其访问和查找变得容易，文件系统使用文件和树形目录的抽象逻辑概念代替了硬盘和光盘等物理设备使用数据块的概念，用户使用文件系统来保存数据不必关心数据实际保存在硬盘（或者光盘）的地址为多少的数据块上，只需要记住这个文件的所属目录和文件名。在写入新数据之前，用户不必关心硬盘上的那个块地址没有被使用，硬盘上的存储空间管理（分配和释放）功能由文件系统自动完成，用户只需要记住数据被写入到了哪个文件中。”

 把其中核心的东西标记出来，即可看出文件系统的本质：一种方便管理、组织、访问数据的软件。

- 对于管理来说，主要是磁盘空闲空间的管理
- 对于组织来说，主要是通过引入文件（inode）、树形目录（dentry）来组织用户的数据。文件包含用户的数据、树形为用户提供了一个对数据进行分类的功能。
- 对于访问来说，通过目录+文件名的方式进行文件创建、删除、读、写（也就是所谓的增、删、查、改）。



在以上核心的功能之上，加入错误异常处理的机制比如通过日志保证操作的原子性以及数据的一致性、通过fsck机制确保文件系统的正确，加入权限控制，加入一些高级特性比如快照、重复数据删除等，加上文件的并发控制等就是一个文件系统了。



## XFS 简介

XFS最早针对IRIX操作系统开发，是一个高性能的日志型文件系统，能够在断电以及操作系统崩溃的情况下保证文件系统数据的一致性。它是一个64位的文件系统，后来进行开源并且移植到了Linux操作系统中，目前CentOS 7将XFS+LVM作为默认的文件系统。据官方所称，XFS对于大文件的读写性能较好。 

性能问题对于大多数文件系统而言，都是一个头疼的事情，特别是在高并发大量小文件这种场景下，而XFS采用了一些好的思想比如引入分配组、B+树、extent等方法来提高性能，这些将在下面介绍。

XFS 官方文档：https://xfs.org/index.php/XFS_Papers_and_Documentation



## 分配组

分配组是XFS抽象程度最高的概念。XFS文件系统内部被分为多个“分配组”，它们是文件系统中的等长线性存储区。每个分配组各自管理自己的inode和剩余空间。文件和文件夹可以跨越分配组。这一机制为XFS提供了可伸缩性和并行特性——多个线程和进程可以同时在同一个文件系统上并行执行I/O操作。这种由分配组带来的内部分区机制在一个文件系统跨越多个物理设备时特别有用，使得优化对下级存储部件的吞吐量利用率成为可能。

在一个磁盘上创建XFS文件系统之后，磁盘会被格式化成如下格式：

![img](../../resource/Center1.png)

在CentOS7上默认的是创建4个AG。每个AG都相当于是1个独立的文件系统，维护着自己的free space以及inode，其主要包括以下信息：

- superblock：描述整个文件系统的信息。
- 空闲空间管理。
- inode的分配和记录管理





## xfs 文件系统挂载选项

官方文档：https://www.kernel.org/doc/html/latest/admin-guide/xfs.html?highlight=fstrim



#### **allocsize=size**

设置延迟分配写出时缓冲的I / O文件结束预分配大小（默认大小为64KiB）。此选项的有效值是页面大小（通常为4KiB）到1GiB（含）之间，以2的幂为单位递增。

默认行为是动态文件末尾预分配大小，它使用一组启发式方法根据文件中的当前分配模式和对文件的访问模式来优化预分配大小。指定固定的allocsize值将关闭动态行为。



#### **attr2 or noattr2**

这些选项启用/禁用将内联扩展属性存储在磁盘上的方式的“机会主义”改进。当第一次选择attr2时使用新格式时（设置或删除扩展属性时），磁盘上超级块功能位字段将更新以反映该格式的使用。

默认行为由磁盘功能位确定，该功能位指示attr2行为处于活动状态。如果设置了任何一个安装选项，则它将成为文件系统使用的新默认值。

启用CRC的文件系统始终使用attr2格式，因此如果设置了noattr2 mount选项，它将拒绝该设置。



#### **discard or nodiscard (default)**

启用/禁用命令发布，以使块设备回收文件系统释放的空间。这对于SSD设备，精简配置的LUN和虚拟机映像很有用，但可能会影响性能。 

注意：当前，建议您使用fstrim应用程序丢弃未使用的块，而不是使用丢弃挂载选项，因为此选项对性能的影响非常严重。

