# Cgroup 实践

linux cgroup全称linux control group ,是linux内核的一个功能，用来限 制、控制与分离一个进程组群的资源（如cpu、内存、磁盘输入输出等）。这个项目最早是由Google的工程师在2006年发起（主要是Paul Menage和Rohit Seth），最早的名称为进程容器（process containers）。在2007年时，因为在Linux内核中，容器（container）这个名词太过广泛，为避免混乱，被重命名为cgroup。

linux把cgroup实现成了一个file system。 默认情况下编译内核时打开cgroup的系统中所有进行位于同一个cgroup，就是根，这个cgroup享有所有的系统资源。 我们可以通过cgroup文件系统建立一个新的cgroup，然后配置这个新的cgroup，配置内容包括为其分配进程，分配资源等。这个创建和分配的过程都是通过cgroup文件系统通过shell echo写进文件系统资源的。

lssubsys –all查看系统中默认挂载的子系统 。

使用 `df -h` 时会看到 Cgroup 文件系统：

```
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs         16G     0   16G   0% /dev
tmpfs            16G     0   16G   0% /dev/shm
tmpfs            16G  688K   16G   1% /run
tmpfs            16G     0   16G   0% /sys/fs/cgroup
/dev/vda2        40G   37G  2.7G  94% /
/dev/vda1       509M  372M  138M  74% /boot
/dev/vde        197G  7.4G  180G   4% /mnt/vde
tmpfs           3.2G     0  3.2G   0% /run/user/1003
```

查看 `/sys/fs` 下的文件系统：

```
$ ls /sys/fs/
bpf  cgroup  ext4  pstore  xfs
```

创建 `/sys/fs/cgroup/memory/test` 目录

```bash
$ sudo mkdir /sys/fs/cgroup/memory/test
```

查看创建的目录：

![image-20200401101131187](../resource/image-20200401101131187.png)

创建目录后，会自动生成这些文件。

其中带 memsw 的表示虚拟内存，不带 memsw 的仅包括物理内存。其中，limit_in_bytes 是用来限制内存使用的，其他的则是统计报告。

**memory.memsw.limit_in_bytes**：内存＋swap空间使用的总量限制。 **memory.limit_in_bytes**：内存使用量限制。

memory.memsw.limit_in_bytes 必须大于或等于 memory.limit_in_byte。

往test这个cgroup中添加进程只要将进程号写入cgroup.procs就可以了。

查看根 Cgroup 的进程 ID 列表：

```
cat /sys/fs/cgroup/memory/cgroup.procs
```

限制内存：

```
echo 64M > /sys/fs/cgroup/memory/test/memory.limit_in_bytes
echo 64M > /sys/fs/cgroup/memory/test/memory.memsw.limit_in_bytes
```





## cgconfig 服务

在 centos7 中，可以安装一个 cgconfig 服务：

```bash
$ sudo yum install libcgroup-tools
$ systemctl enable cgconfig
$ systemctl start cgconfig
$ systemctl status cgconfig
```

通过配置 `/etc/cgconfig.conf` 文件，再重启 cgconfig 服务就可以实现配置 cgroup 了。























