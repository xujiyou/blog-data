# containerd 配置

containerd 的配置文档在 ：https://github.com/containerd/containerd/blob/master/docs/man/containerd-config.toml.5.md

默认配置文件地址：/etc/containerd/config.toml

containerd 的参数 `--config`  决定了配置文件的地址。

#### root

存放数据的根目录，(Default: "/var/lib/containerd")

#### **state**

状态目录 ，(Default: "/run/containerd")

#### **oom_score**

应用于容器式守护进程的内存不足（OOM）分数（默认值：0）

#### **imports**

导入是要包括的其他配置文件的列表。这样可以拆分主配置文件，并分别保留一些部分(例如，供应商可以将自定义运行时配置保存在单独的文件中，而无需修改主config.toml)，

导入的文件将覆盖简单的字段，例如int或string（如果不为空），并将追加数组和映射字段。导入的文件也会进行版本控制，并且版本不能高于主配置。





## **grpc**

 gRPC socket 监听设置，包含三个字段

- **address** (Default: "/run/containerd/containerd.sock")
- **uid** (Default: 0)
- **gid** (Default: 0)



## **debug**

开启和配置 debug socket  监听设置，包含四个字段

- **address** (Default: "/run/containerd/debug.sock")
- **uid** (Default: 0)
- **gid** (Default: 0)
- **level** (Default: "info") sets the debug log level



## **metrics**

监控指标配置

- **address** (Default: "") 指标终结点默认情况下不监听
- **grpc_histogram** (Default: false) 开启或关闭gRPC直方图指标



## **cgroup**

Linux cgroup特定设置部分

**path** (Default: "") 为创建的容器指定自定义cgroup路径



## **plugins**

插件部分包含已安装插件公开的配置选项。默认情况下，以下插件已启用，其设置如下所示。默认情况下未启用的插件将提供其自己的配置值文档。

- **[plugins.cgroup]** has one option **no_prometheus** (Default: **false**)
- **[plugins.diff]** has one option **default**, a list by default set to **["walking"]**
- **[plugins.linux]** has several options for configuring the runtime, shim, and related options: **shim** specifies the shim binary (Default: **"containerd-shim"**), **runtime** is the OCI compliant runtime binary (Default: **"runc"**), **runtime_root** is the root directory used by the runtime (Default: **""**), **no_shim** specifies whether to use a shim or not (Default: **false**), **shim_debug** turns on debugging for the shim (Default: **false**)
- **[plugins.scheduler]** has several options that perform advanced tuning for the scheduler: **pause_threshold** is the maximum amount of time GC should be scheduled (Default: **0.02**), **deletion_threshold** guarantees GC is scheduled after n number of deletions (Default: **0** [not triggered]), **mutation_threshold** guarantees GC is scheduled after n number of database mutations (Default: **100**), **schedule_delay** defines the delay after trigger event before scheduling a GC (Default **"0ms"** [immediate]), **startup_delay** defines the delay after startup before scheduling a GC (Default **"100ms"**)



#### 