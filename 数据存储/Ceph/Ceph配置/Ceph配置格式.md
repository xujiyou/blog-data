# Ceph 基本配置

官方教程：https://ceph.readthedocs.io/en/latest/rados/configuration/ceph-conf/

Ceph 集群中运行的守护进程包括：**Ceph Monitor**（ceph-mon）、**Ceph Manager**（ceph-mgr）、**Ceph OSD Daemon**（ceph-osd）。

如果要运行 Ceph 分布式文件系统，还要加一个 **Ceph Metadata Server**（ceph-mds）守护进程。

如果要支持一些对象储存的接口，比如 S3 API，还需要支持 radosgw

每个守护程序都有一系列配置选项，每个配置选项都有一个默认值。您可以通过更改这些配置选项来调整系统的行为。



## 配置名称

所有Ceph配置选项都有一个唯一的名称，该名称由用小写字母组成的单词和下划线（_）组成。

在命令行上指定选项名称时，下划线（_）或破折号（-）字符可以互换使用（例如--mon-host等效于--mon_host）。

当配置选项名称出现在配置文件中时，也可以使用空格代替下划线或破折号！！！



## 配置源

每个Ceph守护进程，进程和库都将从以下列出的几个来源中提取其配置。如果同时存在，则列表后面的源将覆盖列表前面的源：

- 默认值
- Monitor 集群的集中配置数据库
- 存储在本地主机上的配置文件，比如 /etc/ceph/ceph.conf
- 环境变量
- 命令行参数
- 管理员设置的运行时替代

Ceph进程在启动时要做的第一件事之一就是解析通过命令行，环境和本地配置文件提供的配置选项。然后，该过程将与 Monitor 集群联系，以检索整个集群集中存储的配置。一旦可获得完整的配置视图，则将继续执行守护程序或进程。



#### 启动选项

由于某些配置选项会影响进程与 Monitor 联系，进行身份验证和检索集群存储的配置的能力，因此需要将它们本地存储在节点上并在本地配置文件中进行设置。这些选项包括：

- `mon_host` 配置集群的 monitors 列表
- `mon_dns_serv_name` （默认 *ceph-mon*）DNS SRV记录的名称，以检查以通过DNS识别集群监视器
- `mon_data`, `osd_data`, `mds_data`, `mgr_data` 以及定义守护程序将数据存储在哪个本地目录中的类似选项。
- `keyring`, `keyfile` 或者是 `key` 可以用来指定用于与 monitors 进行身份验证的身份验证凭据。请注意，在大多数情况下，默认密钥环位置在上面指定的数据目录中。

在大多数情况下，这些默认值都是合适的，但 `mon_host` 选项除外，该选项标识群集的 Monitor 的地址。使用 DNS 标识 Monitor 时，可以完全避免使用本地ceph配置文件。



#### 跳过从 Monitor 中获取配置

可以通过--no-mon-config选项传递给任何进程，以跳过从集群监视器检索配置的步骤。在完全通过配置文件管理配置或 Monitor 群集当前已关闭但需要完成一些维护活动的情况下，这很有用。



## 配置段

对于每个配置选项，任何给定的进程或守护程序都有一个值。但是，选项的值可能会在不同的守护程序类型之间变化，甚至是同一类型的守护程序也可能不同。存储在 Monitor 配置数据库或本地配置文件中的Ceph选项分为几部分，以指示它们适用于哪些守护程序或客户端。

所有的段有：

- global：全局下的设置会影响Ceph存储群集中的所有守护程序和客户端。
- mon：mon下的设置会影响Ceph存储群集中的所有ceph-mon守护程序，并覆盖global中的相同设置。
- mgr：mgr部分中的设置会影响Ceph Storage Cluster中的所有ceph-mgr守护程序，并覆盖global中的相同设置。
- osd：osd下的设置会影响Ceph Storage Cluster中的所有ceph-osd守护进程，并覆盖global中的相同设置。
- mds：mds部分中的设置会影响Ceph存储群集中的所有ceph-mds守护程序，并覆盖global中的相同设置。
- client：客户端下的设置会影响所有Ceph客户端（例如已安装的Ceph文件系统，已安装的Ceph块设备等）以及Rados Gateway（RGW）守护程序。

部分还可以指定单个守护程序或客户端名称。例如，mon.foo，osd.123和client.smith都是有效的节名称。

任何给定的守护程序都将从全局部分，守护程序或客户机类型部分以及共享其名称的部分中获取其设置。最具体部分中的设置优先，因此例如，如果在同一源（即，在同一配置文件中）的global，mon和mon.foo中都指定了相同的选项，则将使用mon.foo值。

如果在同一部分中指定了同一配置选项的多个值，则以最后一个值为准。

请注意，本地配置文件中的值始终优先于监视器配置数据库中的值，而不管它们出现在哪个部分中。

比如：

```ini
[global]
debug ms = 0

[osd]
debug ms = 1

[osd.1]
debug ms = 10

[osd.2]
debug ms = 10
```



## 元变量

元变量极大地简化了Ceph存储集群的配置。在配置值中设置了元变量后，Ceph会在使用配置值时将元变量扩展为具体值。Ceph元变量类似于Bash shell中的变量扩展。

Ceph支持以下元变量：

- \$cluster：扩展为Ceph存储群集名称。在同一硬件上运行多个Ceph存储群集时很有用。比如：/etc/ceph/\$cluster.client.admin.keyring，默认值：ceph
- \$type：扩展为守护程序或进程类型（例如mds，osd或mon），例如：/var/lib/ceph/\$type
- \$id：扩展为守护程序或客户端标识符。对于osd.0，该值为0；否则为0。对于mds.a，它将是a。例如：/var/lib/ceph/\$type/\$cluster-$id
- \$host：扩展为运行进程的主机名。
- \$name：扩展为 \$type.\$id，比如/var/run/ceph/\$cluster-\$name.asok
- \$pid：扩展为守护进程pid。比如：/var/run/ceph/\$cluster-\$name-\$pid.asok



## 配置文件

在启动时，Ceph进程在以下位置搜索配置文件：

1. \$CEPH_CONF（即 \$CEPH_CONF 环境变量后的路径）
2. -c path/path (使用 -c 命令行选项指定配置文件)
3. /etc/ceph/$cluster.conf
4. ~/.ceph/$cluster.conf
5. ./$cluster.conf (在当前工作目录中)
6. 在 FreeBSD 系统中：/usr/local/etc/ceph/$cluster.conf

其中 $cluster 是群集的名称（默认ceph）。

Ceph配置文件使用ini样式语法。您可以在注释之前添加井号（＃）或分号（;）。例如：

```ini
# <--A number (#) sign precedes a comment.
; A comment may be anything.
# Comments always follow a semi-colon (;) or a pound (#) on each line.
# The end of the line terminates a comment.
# We recommend that you provide comments in your configuration file(s).
```

#### 选项值

配置选项的值是一个字符串。如果太长而无法容纳在一行中，则可以在行尾添加反斜杠（\）作为行继续标记，因此该选项的值将是当前行中=之后的字符串与该字符串的组合在下一行：

```ini
[global]
foo = long long ago\
long ago
```

在上面的示例中，“ foo”的值为“long long ago long ago”。

通常，选项值以换行符或注释结尾，例如

```ini
[global]
obscure one = difficult to explain # I will try harder in next release
simpler one = nothing to explain
```

如果选项值包含空格，并且我们想使其明确，则可以使用单引号或双引号将其引起来，例如

```ini
[global]
line = "to be, or not to be"
```

某些字符不允许直接出现在选项值中。它们是=，＃、;和[。如果需要，我们需要逃避它们，例如

```ini
[global]
secret = "i love \# and \["
```



## Monitor 配置数据库

Monitor 集管理整个群集可以使用的配置选项数据库，从而可以简化整个系统的中央配置管理。可以并且应该将绝大多数配置选项存储在此处，以简化管理和提高透明度。

少数设置可能仍需要存储在本地配置文件中，因为它们会影响连接到监视器，验证和获取配置信息的能力。在大多数情况下，这仅限于mon_host选项，尽管也可以通过使用DNS SRV记录来避免。



#### 段名字

除了上面说的段名字之外，还可以这样配置：

- type:location：其中 type 是 CRUSH 属性，例如机架或主机，而 location 是该属性的值。例如，host:foo 会将选项限制为仅在特定主机上运行的守护程序或客户端。
- class:device-class：其中 device-class 是 CRUSH 设备类的名称（例如 hdd 或 ssd ）。例如，class:ssd 将仅将选项限制为由 SSD 支持的OSD。（此掩码对非 OSD 守护程序或客户端无效。）

设置配置选项时，段可以是节名，掩码或两者的组合，并用斜杠（/）字符分隔。例如，osd/rack:foo 表示 foo 机架中的所有 OSD 守护程序。

在查看配置选项时，通常将段名称和掩码分隔为单独的字段或列，以简化可读性。



#### 命令

以下命令用于配置集群：

- `ceph config dump` 将显示群集的整个配置数据库。
- `ceph config get <who>` 将显示存储在监视器的配置数据库中的特定守护程序或客户端（例如mds.a）的配置。
- `ceph config set <who> <option> <value>` 将在 Monitor 的配置数据库中设置配置选项。
- `ceph config show <who>` 将显示报告的正在运行的守护程序的运行配置。如果还使用了本地配置文件，或者在命令行或运行时覆盖了选项，则这些设置可能与 Monitor 存储的设置不同。选项值的来源报告为输出的一部分。
- `ceph config assimilate-conf -i <input file> -o <output file>` 将从输入文件中提取配置文件，并将所有有效选项移至 Monitor 的配置数据库中。Monitor 无法识别，无效或无法控制的任何设置都将在输出文件中存储的简短配置文件中返回。此命令对于从旧版配置文件过渡到基于集中式监视器的配置很有用。

其中在使用 `ceph config dump` 和 `ceph config get` 命令前如果没往 Monitor 内写入配置，这俩命令就得不到任何配置，这俩命令只显示 Monitor 数据库中的配置，而 `ceph config show` 会显示所有配置来源的配置。



#### 帮助

可以通过以下方式获得有关特定选项的帮助：

```bash
ceph config help <option>
```

请注意，这将使用编译到正在运行的监视器中的配置架构。如果您有混合版本的集群（例如，在升级过程中），则可能还需要从特定的运行守护程序查询选项模式：

```bash
ceph daemon <name> config help [option]
```

实例：

```bash
$ ceph config help log_file
log_file - path to log file
  (str, basic)
  Default (non-daemon): 
  Default (daemon): /var/log/ceph/$cluster-$name.log
  Can update at runtime: false
  See also: [log_to_file,log_to_stderr,err_to_stderr,log_to_syslog,err_to_syslog]
```

或者：

```bash
$ ceph config help log_file -f json-pretty

{
    "name": "log_file",
    "type": "str",
    "level": "basic",
    "desc": "path to log file",
    "long_desc": "",
    "default": "",
    "daemon_default": "/var/log/ceph/$cluster-$name.log",
    "tags": [],
    "services": [],
    "see_also": [
        "log_to_file",
        "log_to_stderr",
        "err_to_stderr",
        "log_to_syslog",
        "err_to_syslog"
    ],
    "enum_values": [],
    "min": "",
    "max": "",
    "can_update_at_runtime": false,
    "flags": []
}
```

level属性可以是basic，advanced或dev中的任何一个。 dev选项供开发人员使用，通常用于测试目的，不建议操作员使用。



## 运行期改变配置

在大多数情况下，Ceph允许您在运行时更改守护程序的配置。此功能对于增加/减少日志输出，启用/禁用调试设置，甚至是运行时优化非常有用。

一般来说，可以通过ceph config set命令以常规方式更新配置选项。例如，在特定的OSD上启用调试日志级别，请执行以下操作：

```bash
$ ceph config set osd.123 debug_ms 20
```

请注意，如果在本地配置文件中还自定义了相同的选项，则监视器设置将被忽略（其优先级低于本地配置文件）。



#### override 值

还可以使用Ceph CLI上的tell或daemon界面临时设置一个选项。这些替代值是短暂的，因为它们仅影响正在运行的进程，并且在守护程序或进程重新启动时将被丢弃/丢失。

override 级别的值的优先级高于其他的所有配置。

有两种方式可以设置 override 值。

1. ceph tell：

   ```bash
   ceph tell <name> config set <option> <value>
   ```

   例子：

   ```bash
   $ ceph tell osd.123 config set debug_osd 20
   ```

   tell命令还可以接受守护程序标识符的通配符。例如，要调整所有OSD守护程序的调试级别，请执行以下操作：

   ```bash
   $ ceph tell osd.* config set debug_osd 20
   ```

2. 在运行该进程的主机上，我们可以通过 /var/run/ceph 中的套接字直接连接到该进程，其中：

   ```bash
   ceph daemon <name> config set <option> <value>
   ```

   实例：

   ```
   $ ceph daemon osd.4 config set debug_osd 20
   ```

请注意，在 `ceph config show` 命令输出中，将显示这些临时值以及源。





## 查看运行期配置

您可以使用ceph config show命令查看为正在运行的守护程序设置的当前选项。例如，：

```bash
$ ceph config show osd.0
```

将显示该守护程序的（非默认）选项。您还可以通过以下方式查看特定选项：

```bash
$ ceph config show osd.0 debug_osd
```

或使用以下命令查看所有选项（具有默认值的选项）：

```bash
$ ceph config show-with-defaults osd.0
```

也可以通过管理套接字从本地主机连接到正在运行的守护程序来观察其设置。例如，：

```bash
$ ceph daemon osd.0 config show
```

显示和默认配置不同的配置项：

```bash
$ ceph daemon osd.0 config diff
```

仅显示非默认设置（值的来源：配置文件，监视器，替代等）:

```bash
$ ceph daemon osd.0 config get debug_osd
```



## 自从 NAUTILUS 版本以来的改动

当前最新版本是 Octopus，即 15.0.0 以后的版本，2020年4月发行的，上一个版本是 Nautilus，即 14.0.0 以后的版本，2019年2月发行的。

Octopus 的改动如下：

- 允许重复的配置选项，并且不会打印警告。最后一个的值获胜。在此版本之前，我们会在看到重复值的行时打印出警告消息，例如：

  ```
  warning line 42: 'foo' in section 'bar' redefined
  ```

- 无效的UTF-8选项将被警告消息忽略。但是自 Octopus 以来，它们被视为致命错误。
- 反斜杠\用作行继续标记，以将下一行与当前行合并。在使用 Octopus 之前，必须在反斜杠后加上非空行。但是在 Octopus 中，现在允许在反斜杠后使用空行。
- 在配置文件中，每一行都指定一个单独的配置选项。选项的名称及其值用=分隔。值可以用单引号或双引号引起来。但是，如果指定了无效的配置，我们会将其视为无效的配置文件。
- 在Octopus之前，如果在配置文件中未指定任何节名称，则所有选项都将分组到 global 节中。但是现在不鼓励这样做。从 Octopus 开始，对于没有节名称的配置文件，只允许使用一个选项。









