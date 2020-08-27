# Ceph 认证配置

官方文档：https://ceph.readthedocs.io/en/latest/rados/configuration/auth-config-ref/

默认情况下，会启用cephx协议。密码认证有一些计算成本，尽管通常应该很低。如果连接客户端和服务器主机的网络环境非常安全，并且您负担不起身份验证的费用，则可以将其关闭。通常不建议这样做。

> 如果禁用身份验证，则可能会受到中间人攻击，从而更改您的客户端/服务器消息，这可能会导致灾难性的安全影响。



## 部署场景

部署Ceph集群有两种主要方案，它们会影响您最初配置Cephx的方式。大多数首次使用Ceph的用户都使用ceph-deploy创建集群（最简单）。对于使用其他部署工具（例如Chef，Juju，Puppet等）的群集，您将需要使用手动过程或配置部署工具来引导您的监视器。



#### CEPH-DEPLOY

使用ceph-deploy部署集群时，不必手动引导监视器或创建client.admin用户或密钥环。当您执行ceph-deploy new {initial-monitor（s）}时，Ceph将为您创建一个监视器密钥环（仅用于引导监视器），它将为您生成一个初始的Ceph配置文件，其中包含以下身份验证设置，表明Ceph默认启用身份验证：

```ini
auth_cluster_required = cephx
auth_service_required = cephx
auth_client_required = cephx
```

当您执行ceph-deploy mon create-initial时，Ceph将引导初始监视器，并检索一个包含client.admin用户密钥的ceph.client.admin.keyring文件。此外，它还将检索密钥环，使ceph-deploy和ceph-volume实用程序能够准备和激活OSD和元数据服务器。

当执行ceph-deploy admin {node-name}（注意：必须先安装Ceph）时，会将Ceph配置文件和ceph.client.admin.keyring推送到节点的/ etc / ceph目录中。您将能够在该节点的命令行上以root身份执行Ceph管理功能。



#### 手动部署

手动部署集群时，必须手动引导 Monitor 并创建 client.admin 用户和密钥环。要引导 Monitor ，请遵循监视器引导中的步骤。监控程序引导步骤是使用诸如Chef，Puppet，Juju等第三方部署工具时必须执行的逻辑步骤。



## 启用/关闭 cephx

启用Cephx要求您已为 Monitor ，OSD和 MDS 服务器部署了密钥。如果您只是在打开/关闭Cephx上切换，则不必重复启动过程。



#### 开启 Cephx

启用cephx时，Ceph将在默认搜索路径中查找密钥环，其中包括/etc/ceph/$cluster.$name.keyring。可以通过在Ceph配置文件的[global]部分中添加密钥环选项来覆盖此位置，但是不建议这样做。

执行以下过程以在禁用身份验证的群集上启用cephx。如果您（或您的部署实用程序）已经生成了密钥，则可以跳过与生成密钥有关的步骤。

1. 创建一个client.admin 密钥，并为您的客户端主机保存密钥的副本：

   ```bash
   $ ceph auth get-or-create client.admin mon 'allow *' mds 'allow *' mgr 'allow *' osd 'allow *' -o /etc/ceph/ceph.client.admin.keyring
   ```

   >这将破坏任何现有的/etc/ceph/client.admin.keyring文件。如果部署工具已经为您完成了此步骤，则不要执行此步骤。小心！

2. 为您的 Monitor 群集创建一个密钥环，并生成一个 Monitor 密钥：

   ```bash
   $ ceph-authtool --create-keyring /tmp/ceph.mon.keyring --gen-key -n mon. --cap mon 'allow *'
   ```

3. 将监视器密钥环复制到每个监视器的mon数据目录中的ceph.mon.keyring文件中。例如，要将其复制到群集ceph中的mon.a，请使用以下命令：

   ```bash
   $ cp /tmp/ceph.mon.keyring /var/lib/ceph/mon/ceph-a/keyring
   ```

4. 为每个MGR生成一个密钥，其中{$ id}是MGR字母：

   ```bash
   $ ceph auth get-or-create mgr.{$id} mon 'allow profile mgr' mds 'allow *' osd 'allow *' -o /var/lib/ceph/mgr/ceph-{$id}/keyring
   ```

5. 为每个OSD生成一个密钥，其中{$ id}是OSD编号：

   ```bash
   $ ceph auth get-or-create osd.{$id} mon 'allow rwx' osd 'allow *' -o /var/lib/ceph/osd/ceph-{$id}/keyring
   ```

6. 为每个MDS生成一个密钥，其中{$ id}是MDS字母：

   ```bash
   $ ceph auth get-or-create mds.{$id} mon 'allow rwx' osd 'allow *' mds 'allow *' mgr 'allow profile mds' -o /var/lib/ceph/mds/ceph-{$id}/keyring
   ```

7. 通过在Ceph配置文件的[global]部分中设置以下选项来启用cephx身份验证：

   ```ini
   auth cluster required = cephx
   auth service required = cephx
   auth client required = cephx
   ```

8. 启动 Ceph 集群



#### 禁用 cephx

以下过程介绍了如何禁用Cephx。如果您的集群环境相对安全，则可以抵消运行身份验证的计算费用。我们不建议这样做。但是，在设置和/或故障排除过程中暂时禁用身份验证可能会更容易。

1. 通过在Ceph配置文件的[global]部分中设置以下选项来禁用cephx身份验证：

   ```ini
   auth cluster required = none
   auth service required = none
   auth client required = none
   ```

2. 启动或重启集群。





## 配置

#### 启用 cephx 的相关配置

`auth cluster required`

- Description：

  如果启用，则 Ceph Storage Cluster 守护程序（即 ceph-mon，ceph-osd，ceph-mds 和 ceph-mgr）必须相互认证。有效设置为 cephx 或  none 。

- Type：String
- Required：No
- Default：`cephx`.

`auth service required`

- Description

  如果启用，则 Ceph Storage Cluster 守护进程需要 Ceph 客户端向 Ceph Storage Cluster 进行身份验证才能访问Ceph服务。有效设置为 cephx 或 none。

- Type：String

- Required：No

- Default：`cephx`.

`auth client required`

- Description

  如果启用，则 Ceph 客户端需要 Ceph 存储群集向 Ceph 客户端进行身份验证。有效设置为 cephx 或 none。

- Type：String

- Required：No

- Default：`cephx`.



#### KEYS

在启用身份验证的情况下运行 Ceph 时，ceph 管理命令和 Ceph 客户端需要身份验证密钥才能访问 Ceph 存储群集。

将这些密钥提供给 ceph 管理命令和客户端的最常见方法是在 /etc/ceph 目录下包含一个Ceph密钥环。对于使用 ceph-deploy 的 Cuttlefish 和更高版本，文件名通常为 `ceph.client.admin.keyring`（或`$cluster.client.admin.keyring`）。如果将密钥环包含在 /etc/ceph 目录下，则无需在 Ceph 配置文件中指定密钥环条目。

我们建议将 Ceph Storage Cluster 的密钥环文件复制到要运行管理命令的节点，因为它包含 client.admin 密钥。

您可以使用 ceph-deploy admin 执行此任务。有关详细信息，请参见创建管理主机。要手动执行此步骤，请执行以下操作：

```bash
$ sudo scp {user}@{ceph-cluster-host}:/etc/ceph/ceph.client.admin.keyring /etc/ceph/ceph.client.admin.keyring
```

> 确保ceph.keyring文件在客户端计算机上具有适当的权限集（例如chmod 644）。

可以使用密钥设置（不推荐）在 Ceph 配置文件中指定密钥本身，或者使用密钥文件设置在密钥文件中指定路径。

`keyring`

- Description：密钥环文件的路径。
- Type：String
- Required：No
- Default：`/etc/ceph/$cluster.$name.keyring,/etc/ceph/$cluster.keyring,/etc/ceph/keyring,/etc/ceph/keyring.bin`

`keyfile`

- Description：密钥文件（即仅包含密钥的文件）的路径。
- Type：String
- Required：No
- Default：None

`key`

- Description：键（即键本身的文本字符串）。不建议。
- Type：String
- Required：No
- Default：None



`keyring` 中即包含密钥，又包含权限。



#### 守护进程的密钥环

管理用户或部署工具（例如ceph-deploy）可以与生成用户密钥环的方式相同的方式生成守护程序密钥环。默认情况下，Ceph将守护程序密钥环存储在其数据目录中。默认密钥环位置以及守护程序运行所需的功能如下所示。

`ceph-mon`

- Location：`$mon_data/keyring`
- Capabilities：`mon 'allow *'`

`ceph-osd`

- Location：`$osd_data/keyring`
- Capabilities：`mgr 'allow profile osd' mon 'allow profile osd' osd 'allow *'`

`ceph-mds`

- Location：`$mds_data/keyring`
- Capabilities：`mds 'allow' mgr 'allow profile mds' mon 'allow profile mds' osd 'allow rwx'`

`ceph-mgr`

- Location：`$mgr_data/keyring`
- Capabilities：`mon 'allow profile mgr' mds 'allow *' osd 'allow *'`

`radosgw`

- Location：`$rgw_data/keyring`
- Capabilities：`mon 'allow rwx' osd 'allow rwx'`

> 监视器密钥环（即 mon ）包含密钥但不具有功能，并且不属于集群auth数据库。



#### 签名

Ceph 执行签名检查，以提供一些有限的保护，以防止消息传输中被篡改（例如，“中间人”攻击）。

与 Ceph 身份验证的其他部分一样，Ceph 提供了细粒度的控制，因此您可以为客户端和 Ceph 之间的服务消息启用/禁用签名，并且可以为Ceph 守护程序之间的消息启用/禁用签名。

请注意，即使启用了签名，也不会在传输中对数据进行加密。

`cephx require signatures`

- Description

  如果设置为true，则 Ceph 要求在 Ceph 客户端和 Ceph 存储群集之间以及组成 Ceph 存储群集的守护程序之间的所有消息通信上签名。

- Type：Boolean

- Required：No

- Default：`false`

`cephx cluster require signatures`

- Description

  如果设置为 true，则 Ceph 要求在组成 Ceph 存储群集的 Ceph 守护程序之间的所有消息流量上签名。

- Type：Boolean

- Required：No

- Default：`false`

`cephx service require signatures`

- Description

  如果设置为 true，则 Ceph 要求在 Ceph 客户端和 Ceph 存储集群之间的所有消息流量上签名。

- Type：Boolean

- Required：No

- Default：`false`

`cephx sign messages`

- Description

  如果Ceph版本支持消息签名，则Ceph将对所有消息进行签名，因此更难以欺骗。

- Type：Boolean

- Default：`true`



#### 存活时间

`auth service ticket ttl`

- Description

  当 Ceph 存储群集向 Ceph 客户端发送用于身份验证的票证时，Ceph 存储群集将为该票证分配生存时间。

- Type：Double

- Default：`60*60`



























