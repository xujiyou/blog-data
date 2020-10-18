# YARN 资源配置

官方文档：https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/ResourceModel.html

YARN支持可扩展的资源模型。默认情况下，YARN跟踪所有节点，应用程序和队列的CPU和内存，但是资源定义可以扩展为包括任意“可数”资源。可计数资源是在容器运行时消耗但随后释放的资源。 CPU和内存都是可数的资源。其他示例包括GPU资源和软件许可证。

此外，YARN还支持使用“资源配置文件”，这使用户可以通过单个配置文件指定多个资源请求，类似于Amazon Web Services Elastic Compute Cluster实例类型。例如，“大”可能意味着8个虚拟内核和16GB RAM。



## 配置

yarn-site.xml

| Configuration Property                           | Description                                 |
| :----------------------------------------------- | :------------------------------------------ |
| `yarn.resourcemanager.resource-profiles.enabled` | 指示是否启用资源配置文件支持。默认为false。 |

resource-types.xml

| Configuration Property                              | Description                                                  |
| :-------------------------------------------------- | :----------------------------------------------------------- |
| `yarn.resource-types`                               | 以逗号分隔的其他资源列表。可能不包含memory，memory-mb或vcores |
| `yarn.resource-types.<resource>.units`              | 指定资源类型的默认单位                                       |
| `yarn.resource-types.<resource>.minimum-allocation` | 指定资源类型的最小请求                                       |
| `yarn.resource-types.<resource>.maximum-allocation` | 指定资源类型的最大请求                                       |

node-resources.xml

| Configuration Property                      | Description                      |
| :------------------------------------------ | :------------------------------- |
| `yarn.nodemanager.resource-type.<resource>` | 节点管理器中可用的指定资源的计数 |

请注意，如果使用resource-types.xml和node-resources.xml文件，它们也需要与yarn-site.xml放在同一配置目录中。或者，可以将属性放置在yarn-site.xml文件中。



## YARN Resource Model

#### 资源管理器

资源管理器是跟踪集群中哪些资源的最终仲裁者。资源管理器从XML配置文件加载其资源定义。例如，要定义除CPU和内存之外的新资源，应配置以下属性：

```xml
<configuration>
  <property>
    <name>yarn.resource-types</name>
    <value>resource1,resource2</value>
    <description>
    The resources to be used for scheduling. Use resource-types.xml
    to specify details about the individual resource types.
    </description>
  </property>
</configuration>
```

有效的资源名称必须以字母开头，并且只能包含字母，数字和以下任何一个：“。”，“ _”或“-”。有效的资源名称还可以可选地在名称空间之后加上斜杠。有效的名称空间由句点分隔的字母，数字和破折号组成。例如，以下是有效的资源名称：

- myresource
- my_resource
- My-Resource01
- com.acme/myresource

以下是无效资源名称的示例：

- 10myresource
- my resource
- com/acme/myresource
- $NS/myresource
- -none-/myresource

对于定义的每种新资源类型，可以添加可选的单位属性以设置资源类型的默认单位。有效值为：

| Unit Name | Meaning                  |
| :-------- | :----------------------- |
| p         | pico                     |
| n         | nano                     |
| u         | micro                    |
| m         | milli                    |
|           | default, i.e. no unit    |
| k         | kilo                     |
| M         | mega                     |
| G         | giga                     |
| T         | tera                     |
| P         | peta                     |
| Ki        | binary kilo, i.e. 1024   |
| Mi        | binary mega, i.e. 1024^2 |
| Gi        | binary giga, i.e. 1024^3 |
| Ti        | binary tera, i.e. 1024^4 |
| Pi        | binary peta, i.e. 1024^5 |



该属性必须命名为yarn.resource-types.<resource> .units。每个定义的资源还可以具有可选的最小和最大属性。这些属性必须分别命名为yarn.resource-types.<resource> .minimum-allocation和yarn.resource-types.<resource> .maximum-allocation。

yarn.resource-types属性以及任何单位，最小或最大属性都可以在通常的yarn-site.xml文件或名为resource-types.xml的文件中定义。例如，以下内容可能会出现在两个文件中：

```xml
<configuration>
  <property>
    <name>yarn.resource-types</name>
    <value>resource1, resource2</value>
  </property>

  <property>
    <name>yarn.resource-types.resource1.units</name>
    <value>G</value>
  </property>

  <property>
    <name>yarn.resource-types.resource2.minimum-allocation</name>
    <value>1</value>
  </property>

  <property>
    <name>yarn.resource-types.resource2.maximum-allocation</name>
    <value>1024</value>
  </property>
</configuration>
```



#### 节点管理器

每个节点管理器独立定义该节点可用的资源。通过为每个可用资源设置属性来完成资源定义。该属性必须命名为yarn.nodemanager.resource-type.<resource>，并且可以放置在常规的yarn-site.xml文件或名为noderesources.xml的文件中。该属性的值应为节点提供的资源量。例如：

```xml
<configuration>
 <property>
   <name>yarn.nodemanager.resource-type.resource1</name>
   <value>5G</value>
 </property>

 <property>
   <name>yarn.nodemanager.resource-type.resource2</name>
   <value>2m</value>
 </property>

</configuration>
```

请注意，用于这些资源的单位不必与资源管理器所保存的定义相匹配。如果单位不匹配，资源管理器将自动进行转换。





#### MapReduce 使用资源

MapReduce向YARN请求三种不同的容器：应用程序主容器，Map 容器和 reduce 容器。对于每种容器类型，都有一组对应的属性可用于设置请求的资源。

在MapReduce中设置资源请求的属性为：

| Property                                    | Description                                                  |
| :------------------------------------------ | :----------------------------------------------------------- |
| `yarn.app.mapreduce.am.resource.mb`         | Sets the memory requested for the application master container to the value in MB. No longer preferred. Use `yarn.app.mapreduce.am.resource.memory-mb` instead. Defaults to 1536. |
| `yarn.app.mapreduce.am.resource.memory`     | Sets the memory requested for the application master container to the value in MB. No longer preferred. Use `yarn.app.mapreduce.am.resource.memory-mb` instead. Defaults to 1536. |
| `yarn.app.mapreduce.am.resource.memory-mb`  | Sets the memory requested for the application master container to the value in MB. Defaults to 1536. |
| `yarn.app.mapreduce.am.resource.cpu-vcores` | Sets the CPU requested for the application master container to the value. No longer preferred. Use `yarn.app.mapreduce.am.resource.vcores` instead. Defaults to 1. |
| `yarn.app.mapreduce.am.resource.vcores`     | Sets the CPU requested for the application master container to the value. Defaults to 1. |
| `yarn.app.mapreduce.am.resource.<resource>` | Sets the quantity requested of `<resource>` for the application master container to the value. If no unit is specified, the default unit for the resource is assumed. See the section on units above. |
| `mapreduce.map.memory.mb`                   | Sets the memory requested for the all map task containers to the value in MB. No longer preferred. Use `mapreduce.map.resource.memory-mb` instead. Defaults to 1024. |
| `mapreduce.map.resource.memory`             | Sets the memory requested for the all map task containers to the value in MB. No longer preferred. Use `mapreduce.map.resource.memory-mb` instead. Defaults to 1024. |
| `mapreduce.map.resource.memory-mb`          | Sets the memory requested for the all map task containers to the value in MB. Defaults to 1024. |
| `mapreduce.map.cpu.vcores`                  | Sets the CPU requested for the all map task containers to the value. No longer preferred. Use `mapreduce.map.resource.vcores` instead. Defaults to 1. |
| `mapreduce.map.resource.vcores`             | Sets the CPU requested for the all map task containers to the value. Defaults to 1. |
| `mapreduce.map.resource.<resource>`         | Sets the quantity requested of `<resource>` for the all map task containers to the value. If no unit is specified, the default unit for the resource is assumed. See the section on units above. |
| `mapreduce.reduce.memory.mb`                | Sets the memory requested for the all reduce task containers to the value in MB. No longer preferred. Use `mapreduce.reduce.resource.memory-mb` instead. Defaults to 1024. |
| `mapreduce.reduce.resource.memory`          | Sets the memory requested for the all reduce task containers to the value in MB. No longer preferred. Use `mapreduce.reduce.resource.memory-mb` instead. Defaults to 1024. |
| `mapreduce.reduce.resource.memory-mb`       | Sets the memory requested for the all reduce task containers to the value in MB. Defaults to 1024. |
| `mapreduce.reduce.cpu.vcores`               | Sets the CPU requested for the all reduce task containers to the value. No longer preferred. Use `mapreduce.reduce.resource.vcores` instead. Defaults to 1. |
| `mapreduce.reduce.resource.vcores`          | Sets the CPU requested for the all reduce task containers to the value. Defaults to 1. |
| `mapreduce.reduce.resource.<resource>`      | Sets the quantity requested of `<resource>` for the all reduce task containers to the value. If no unit is specified, the default unit for the resource is assumed. See the section on units above. |

请注意，YARN可以修改这些资源请求，以满足配置的最小和最大资源值，或者是配置的增量的倍数。请参见yarn.scheduler.maximum-allocation-mb，yarn.scheduler.minimum-allocation-mb，yarn.scheduler.increment-allocation-mb，yarn.scheduler.maximum-allocation-vcore，yarn.scheduler.minimum-allocation- YARN调度程序配置中的vcores和yarn.scheduler.increment-allocation-vcores属性。



## Resource Profiles

资源配置文件为用户提供了一种使用单个配置文件请求一组资源的简便方法，并为管理员提供了一种调节资源消耗方式的方法。

要配置资源类型，管理员必须在资源管理器的yarn-site.xml文件中将yarn.resourcemanager.resource-profiles.enabled设置为true。该文件定义了受支持的配置文件。例如：

```json
{
    "small": {
        "memory-mb" : 1024,
        "vcores" : 1
    },
    "default" : {
        "memory-mb" : 2048,
        "vcores" : 2
    },
    "large" : {
        "memory-mb": 4096,
        "vcores" : 4
    },
    "compute" : {
        "memory-mb" : 2048,
        "vcores" : 2,
        "gpu" : 1
    }
}
```

在此示例中，用户可以访问具有不同资源设置的四个配置文件。请注意，在计算配置文件中，管理员已如上所述配置了其他资源。

当前，分布式 Sehll 程序是唯一支持资源配置文件的客户端。使用分布式 Shell 程序，用户可以指定资源配置文件名称，该名称将自动转换为一组适当的资源请求。 例如：

```bash
$ hadoop job $DISTSHELL -jar $DISTSHELL -shell_command run.sh -container_resource_profile small
```

































