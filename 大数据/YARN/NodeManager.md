# NodeManager

官方文档：https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/NodeManager.html

NodeManager 负责启动和管理节点上的容器。容器执行AppMaster指定的任务。



## 健康检查

NodeManager 运行服务以确定在其上执行的节点的运行状况。这些服务在磁盘上执行检查以及任何用户指定的测试。如果任何运行状况检查失败，则NodeManager会将节点标记为运行状况不佳，并将其传达给ResourceManager，然后ResourceManager停止为该节点分配容器。节点状态的通信是 NodeManager 和 ResourceManager 之间心跳的一部分。磁盘检查器和运行状况监视器（以下描述）的运行间隔不会影响心跳间隔。心跳发生时，两种检查的状态都用于确定节点的运行状况。

#### 磁盘检查

磁盘检查器检查NodeManager配置为使用的磁盘的状态（本地目录和日志目录，分别使用 `yarn.nodemanager.local-dirs` 和 `yarn.nodemanager.log-dir`s 配置）。检查包括权限和可用磁盘空间。它还会检查文件系统是否处于只读状态。默认情况下，检查每隔2分钟运行一次，但可以将其配置为根据用户需要的频率运行。如果磁盘未通过检查，则NodeManager会停止使用该特定磁盘，但仍将节点状态报告为运行状况良好。但是，如果许多磁盘未通过检查（可以配置该数目，如下所述），则该节点将被报告为不正常，并发送给ResourceManager，并且不会将新容器分配给该节点。

以下配置参数可用于修改磁盘检查：

| Configuration Name                                           | Allowed Values      | Description                                                  |
| :----------------------------------------------------------- | :------------------ | :----------------------------------------------------------- |
| `yarn.nodemanager.disk-health-checker.enable`                | true, false         | Enable or disable the disk health checker service            |
| `yarn.nodemanager.disk-health-checker.interval-ms`           | Positive integer    | The interval, in milliseconds, at which the disk checker should run; the default value is 2 minutes |
| `yarn.nodemanager.disk-health-checker.min-healthy-disks`     | Float between 0-1   | The minimum fraction of disks that must pass the check for the NodeManager to mark the node as healthy; the default is 0.25 |
| `yarn.nodemanager.disk-health-checker.max-disk-utilization-per-disk-percentage` | Float between 0-100 | The maximum percentage of disk space that may be utilized before a disk is marked as unhealthy by the disk checker service. This check is run for every disk used by the NodeManager. The default value is 90 i.e. 90% of the disk can be used. |
| `yarn.nodemanager.disk-health-checker.min-free-space-per-disk-mb` | Integer             | The minimum amount of free space that must be available on the disk for the disk checker service to mark the disk as healthy. This check is run for every disk used by the NodeManager. The default value is 0 i.e. the entire disk can be used. |



#### 外部健康检查脚本

用户可以指定自己的运行状况检查器脚本，这些脚本将由运行状况检查器服务调用。用户可以指定超时以及要传递给脚本的选项。如果脚本超时，则引发异常或输出以字符串ERROR开头的行，该节点被标记为不正常。请注意：

- 0以外的退出代码不被视为失败，因为它可能是由语法错误引起的。因此，该节点不会被标记为不正常。
- 如果脚本由于权限或路径错误等原因而无法执行，则视为失败，并且该节点将被报告为不正常。
- 指定运行状况检查脚本不是强制性的。如果未指定脚本，则仅使用磁盘检查器状态来确定节点的运行状况。

用户最多可以指定4个脚本来使用yarn.nodemanager.health-checker.scripts配置单独运行。还可以为所有脚本配置这些选项（全局配置）：

| Configuration Name                            | Allowed Values   | Description                                                  |
| :-------------------------------------------- | :--------------- | :----------------------------------------------------------- |
| `yarn.nodemanager.health-checker.script`      | String           | The keywords for the health checker scripts separated by a comma. The default is “script”. |
| `yarn.nodemanager.health-checker.interval-ms` | Positive integer | The interval, in milliseconds, at which health checker service runs; the default value is 10 minutes. |
| `yarn.nodemanager.health-checker.timeout-ms`  | Positive integer | The timeout for the health script that’s executed; the default value is 20 minutes. |

可以为每个运行状况检查器脚本设置以下选项。 ％s符号将替换为yarn.nodemanager.health-checker.script中提供的每个关键字。

| Configuration Name                               | Allowed Values   | Description                                                  |
| :----------------------------------------------- | :--------------- | :----------------------------------------------------------- |
| `yarn.nodemanager.health-checker.%s.path`        | String           | Absolute path to the health check script to be run. Mandatory argument for each script. |
| `yarn.nodemanager.health-checker.%s.opts`        | String           | Arguments to be passed to the script when the script is executed. Mandatory argument for each script. |
| `yarn.nodemanager.health-checker.%s.interval-ms` | Positive integer | The interval, in milliseconds, at which health checker service runs. |
| `yarn.nodemanager.health-checker.%s.timeout-ms`  | Positive integer | The timeout for the health script that’s executed.           |

无需指定时间间隔和超时选项。在这种情况下，将使用全局配置。



## NodeManager 重启

本文档概述了NodeManager（NM）重新启动，此功能使NodeManager能够重新启动而不会丢失在节点上运行的活动容器。在较高级别上，NM在处理容器管理请求时会将所有必要状态存储到本地状态存储中。 NM重新启动时，它将通过以下方式恢复：首先加载各种子系统的状态，然后让这些子系统使用已加载状态执行恢复。

启用 NM 重启的步骤：

- 步骤1.要启用NM Restart功能，请将 conf/yarn-site.xml 中的 yarn.nodemanager.recovery.enabled 属性设置为true。

- 步骤2.配置本地文件系统目录的路径，NodeManager可以在其中保存其运行状态。配置为 yarn.nodemanager.recovery.dir

- 步骤3：在恢复状态下启用NM监视，以防止在NM退出时清理正在运行的容器。

  | Property                               | Description                                                  |
  | :------------------------------------- | :----------------------------------------------------------- |
  | `yarn.nodemanager.recovery.supervised` | 如果启用，则运行的NodeManager会在退出时不会尝试清理容器，并假设它将立即重新启动并恢复容器。默认值设置为“ false”。 |

- 步骤4.为NodeManager配置有效的RPC地址。

  | Property                   | Description                                                  |
  | :------------------------- | :----------------------------------------------------------- |
  | `yarn.nodemanager.address` | 临时端口（默认为端口0）不能用于通过yarn.nodemanager.address指定的NodeManager的RPC服务器，因为它可以使NM在重启前后使用不同的端口。这将破坏重新启动之前与NM通信的所有先前运行的客户端。明确将yarn.nodemanager.address设置为具有特定端口号的地址（例如0.0.0.0:45454）是启用NM重新启动的前提条件。 |

- 步骤5.辅助服务
  - 可以将YARN群集中的NodeManager配置为运行辅助服务。对于完全正常运行的NM重新启动，YARN依赖于配置为也支持恢复的任何辅助服务。这通常包括（1）避免使用临时端口，以便先前运行的客户端（在这种情况下，通常是容器）在重新启动后不会受到干扰；（2）通过在NodeManager重新启动并重新初始化时重新加载任何先前状态，使辅助服务本身支持可恢复性。
  - 上面的一个简单示例是MapReduce（MR）的辅助服务“ ShuffleHandler”。 ShuffleHandler已经满足了以上两个要求，因此用户/管理员无需执行任何操作即可支持NM重新启动：（1）配置属性mapreduce.shuffle.port控制NodeManager主机上ShuffleHandler绑定到的端口，以及它默认为非临时端口。 （2）NM重新启动后，ShuffleHandler服务还已经支持恢复先前状态。
  - 有两种方法可通过清单或配置来配置辅助服务。仅当未启用辅助服务清单时，才通过使用配置属性的先前方法来加载辅助服务。使用清单的一个优点是，NM可以根据清单的更改动态地重新加载辅助服务。为了支持重新加载，AuxiliaryService实现必须执行服务停止阶段所需的任何清理，以使NM能够创建辅助服务的新实例。



## 辅助服务类路径隔离

YARN NM 的辅助服务在 HDP 中是有配置的。

要在NodeManager上启动辅助服务，用户必须直接将其jar添加到NodeManager的类路径中，然后将其放在系统类加载器中。但是，如果插件路径上存在多个版本的插件，则无法控制实际加载哪个版本。或者，如果辅助服务引入的依赖项与NodeManager本身之间存在任何冲突，则它们可能会破坏NodeManager，辅助服务或同时破坏两者。为了解决这个问题，我们可以使用不同于系统类加载器的类加载器实例化辅助服务。

本节介绍用于辅助服务类路径隔离的辅助服务清单。要使用清单，必须在yarn-site.xml中将属性yarn.nodemanager.aux-services.manifest.enabled设置为true。

要从文件系统加载清单文件，请在yarn-node.agemanager.aux-services.manifest属性下的yarn-site.xml中设置文件路径。 NM将按yarn.nodemanager.aux-services.manifest.reload-ms指定的时间间隔检查此文件是否有新修改（默认为0；设置时间间隔<= 0表示将不会自动重新加载）。或者，可以通过对端点 http://nm-http-address:port/ws/v1/node/auxiliaryservices进行PUT调用，通过REST API将清单文件发送到NM。请注意，这仅更新一个NM上的清单。读取新清单后，NM将根据清单中找到的服务名称和版本，根据需要添加，删除或重新加载辅助服务。

下面是为CustomAuxService配置类路径隔离的示例清单。可以指定一个或多个文件来构成服务的类路径，并支持jar或存档格式。

```json
{
  "services": [
    {
      "name": "mapreduce_shuffle",
      "version": "2",
      "configuration": {
        "properties": {
          "class.name": "org.apache.hadoop.mapred.ShuffleHandler",
          "mapreduce.shuffle.transfer.buffer.size": "102400",
          "mapreduce.shuffle.port": "13562"
        }
      }
    },
    {
      "name": "CustomAuxService",
      "version": "1",
      "configuration": {
        "properties": {
          "class.name": "org.aux.CustomAuxService"
        },
        "files": [
          {
            "src_file": "${remote-dir}/CustomAuxService.jar",
            "type": "STATIC"
          },
          {
            "src_file": "${remote-dir}/CustomAuxService.tgz",
            "type": "ARCHIVE"
          }
        ]
      }
    }
  ]
}
```



辅助服务配置：

| Configuration Name                                  | Description                                                  |
| :-------------------------------------------------- | :----------------------------------------------------------- |
| `yarn.nodemanager.aux-services.%s.classpath`        | Provide local directory which includes the related jar file as well as all the dependencies’ jar file. We could specify the single jar file or use ${local_dir_to_jar}/* to load all jars under the dep directory. |
| `yarn.nodemanager.aux-services.%s.remote-classpath` | Provide remote absolute or relative path to jar file(We also support zip, tar.gz, tgz, tar and gz files as well). For the same aux-service class, we can only specify one of the configurations: yarn.nodemanager.aux-services.%s.classpath or yarn.nodemanager.aux-services.%s.remote-classpath. The YarnRuntimeException will be thrown. Please also make sure that the owner of the jar file must be the same as the NodeManager user and the permbits should satisfy (permbits & 0022)==0 (such as 600, it’s not writable by group or other). |
| `yarn.nodemanager.aux-services.%s.system-classes`   | Normally, we do not need to set this configuration. The class would be loaded from customized classpath if it does not belongs to system-classes. For example, by default, the package org.apache.hadoop is in the system-classes, if your class CustomAuxService is in the package org.apache.hadoop, it would not be loaded from customized classpath. To solve this, either we could change the package for CustomAuxService, or configure our own system-classes which exclude org.apache.hadoop. |

配置示例：

```xml
<property>
	<name>yarn.nodemanager.aux-services</name>
	<value>mapreduce_shuffle,CustomAuxService</value>
</property>

<property>
	<name>yarn.nodemanager.aux-services.CustomAuxService.classpath</name>
	<value>${local_dir_to_jar}/CustomAuxService.jar</value>
</property>

<!--
<property>
	<name>yarn.nodemanager.aux-services.CustomAuxService.remote-classpath</name>
	<value>${remote-dir_to_jar}/CustomAuxService.jar</value>
</property>
-->

<property>
	<name>yarn.nodemanager.aux-services.CustomAuxService.class</name>
	<value>org.aux.CustomAuxService</value>
</property>

<property>
	<name>yarn.nodemanager.aux-services.mapreduce_shuffle.class</name>
	<value>org.apache.hadoop.mapred.ShuffleHandler</value>
</property>
```

















