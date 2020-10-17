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

















