# Ceph 用户管理

官方文档：https://ceph.readthedocs.io/en/latest/rados/operations/user-management/

本文档介绍了 Ceph 客户端用户及其在 Ceph 存储群集中的身份验证和授权。用户是使用 Ceph 客户端与 Ceph 存储群集守护程序进行交互的个人或系统参与者（例如应用程序）。

当 Ceph 在启用身份验证和授权的情况下运行（默认情况下启用）时，必须指定用户名和包含指定用户的私钥的密钥环（通常是通过命令行）。如果您未指定用户名，则 Ceph 将使用 `client.admin` 作为默认用户名。如果您未指定密钥环，则 Ceph 将通过 Ceph 配置中的密钥环设置来寻找密钥环。例如，如果执行 ceph health 命令而不指定用户或密钥环：

```bash
$ ceph health
```

完整命令如下：

```bash
$ ceph -n client.admin --keyring=/etc/ceph/ceph.client.admin.keyring health
```

或者，您可以使用 `CEPH_ARGS` 环境变量来避免再次输入用户名和密码。

无论 Ceph 客户端的类型如何（例如，块设备，对象存储，文件系统，本机 API 等），Ceph 都将所有数据存储为对象在 Pool 中。Ceph 用户必须有权访问 Pool 才能读取和写入数据。此外，Ceph 用户必须具有执行权限才能使用 Ceph 的管理命令。以下概念将帮助您了解Ceph用户管理。



## 用户

用户可以是个人，也可以是系统参与者，例如应用程序。通过创建用户，您可以控制哪些人（或什么人）可以访问您的Ceph Storage Cluster，以及 Pool 中的数据。

Ceph 具有用户类型的概念。为了用户管理的目的，类型将始终是 client。Ceph以句点（.）分隔的形式标识用户，包括用户类型和用户ID：例如 TYPE.ID，client.admin 或 client.user1。用户加入类型的原因是 Ceph Mon，OSD和元数据服务器也使用 Cephx 协议，但它们不是客户端。区分用户类型有助于区分客户端用户和其他用户-简化访问控制，用户监控和可追溯性。

有时 Ceph 的用户类型可能看起来令人困惑，因为 Ceph 命令行允许您根据命令行的使用情况指定使用或不使用类型的用户。如果指定 `--use`r或`--id`，则可以省略类型。因此，client.user1 可以简单地作为 user1 输入。如果指定 `--name` 或 `-n` ，则必须指定类型和名称，例如 client.user1 。建议尽可能使用类型和名称作为最佳实践。

> Ceph 存储群集用户与 Ceph 对象存储用户或 Ceph 文件系统用户不同。Ceph 对象网关使用 Ceph 存储群集用户在网关守护程序和存储群集之间进行通信，但是网关对最终用户具有自己的用户管理功能。Ceph文件系统使用POSIX语义。与Ceph文件系统关联的用户空间与Ceph存储集群用户不同。



## 授权

Ceph使用术语“capabilities”（caps）来描述授权经过身份验证的用户行使监视器，OSD和元数据服务器的功能。Capabilities 还可以根据其应用程序标记来限制对池中的数据，池中的命名空间或一组池的访问。Ceph管理用户在创建或更新用户时设置用户的 capabilities。

capabilities 语法遵循以下形式：

```
{daemon-type} '{cap-spec}[, {cap-spec} ...]'
```

- **Monitor Caps:** Monitor 权限包括r，w，x访问设置或 profile {name}。例如：

  ```bash
  mon 'allow {access-spec} [network {network/prefix}]'
  
  mon 'profile {name}'
  ```

  {access-spec} 的语法是：

  ```bash
  * | all | [r][w][x]
  ```

  可选的 {network/prefix} 是 CIDR 表示法中的标准网络名称和前缀长度（例如10.3.0.0/16）。如果存在，则此功能的使用仅限于从该网络连接的客户端。

- **OSD Caps:** 包括 `r`, `w`, `x`, `class-read`, `class-write` 权限，或 `profile {name}`，此外，OSD功能还允许 Pool 和名称空间设置。

  ```bash
  osd 'allow {access-spec} [{match-spec}] [network {network/prefix}]'
  osd 'profile {name} [pool={pool-name} [namespace={namespace-name}]] [network {network/prefix}]'
  ```

   `{access-spec}` 语法：

  ```bash
  * | all | [r][w][x] [class-read] [class-write]
  class {class name} [{method name}]
  ```

   `{match-spec}`语法：

  ```bash
  pool={pool-name} [namespace={namespace-name}] [object_prefix {prefix}]
  [namespace={namespace-name}] tag {application} {key}={value}
  ```

- **Manager Caps:** 包括  `r`, `w`, `x`  权限，或`profile {name}`

  ```bash
  mgr 'allow {access-spec} [network {network/prefix}]'
  mgr 'profile {name} [{key1} {match-type} {value1} ...] [network {network/prefix}]'
  ```

  还可以为特定命令，内置管理器服务导出的所有命令或特定附加模块导出的所有命令指定管理器功能。例如：

  ```bash
  mgr 'allow command "{command-prefix}" [with {key1} {match-type} {value1} ...] [network {network/prefix}]'
  mgr 'allow service {service-name} {access-spec} [network {network/prefix}]'
  mgr 'allow module {module-name} [with {key1} {match-type} {value1} ...] {access-spec} [network {network/prefix}]'
  ```

  `{access-spec}` 语法：

  ```
  * | all | [r][w][x]
  ```

  `{service-name}` 是其中之一：

  ```
  mgr | osd | pg | py
  ```

  `{match-type}` 是其中之一：

  ```
  = | prefix | regex
  ```

- **Metadata Server Caps:** 对管理员来说，使用 `allow *` 即可 ，对于其他用户，参考  [CephFS Client Capabilities](https://ceph.readthedocs.io/en/latest/cephfs/client-auth/)

>  Ceph对象网关守护程序（radosgw）是Ceph存储群集的客户端，因此它不表示为Ceph存储群集守护程序类型。

以下条目描述了每种访问功能。



#### allow

权限前缀

#### r

授予用户读取访问权限。监视器必需以检索 CRUSH Map。

#### w

授予用户对对象的写访问权限。

#### x

使用户能够调用类方法（即读取和写入）并在监视器上执行身份验证操作。

#### class-read

使用户能够调用类读取方法。 x的子集。

#### class-write

使用户能够调用类写入方法。 x的子集。

#### `*`, `all`

为用户提供对特定守护程序/池的读取，写入和执行权限，以及执行管理命令的能力。

#### `profile osd` (Monitor only)

授予用户作为 OSD 连接到其他 OSD 或 Mon 的权限。授予 OSD 以使 OSD 能够处理复制心跳流量和状态报告。

#### `profile mds` (Monitor only)

授予用户权限，以将其作为MDS连接到其他MDS或监视器。

#### `profile bootstrap-osd` (Monitor only)

授予用户引导OSD的权限。授予部署工具（如ceph-volume，ceph-deploy等）的权限，以便它们在引导OSD时具有添加密钥等的权限。

#### `profile bootstrap-mds` (Monitor only)

向用户授予引导元数据服务器的权限。赋予部署工具（如ceph-deploy等），因此它们在引导元数据服务器时有权添加密钥等。

#### `profile bootstrap-rbd` (Monitor only)

授予用户引导RBD用户的权限。授予部署工具（如ceph-deploy等）的权限，因此当引导RBD用户时，它们具有添加密钥等的权限。

#### `profile bootstrap-rbd-mirror` (Monitor only)

向用户授予引导rbd-mirror守护程序用户的权限。授予部署工具（如ceph-deploy等）的权限，因此它们在引导rbd-mirror守护程序时有权添加密钥等。

#### `profile rbd` (Manager, Monitor, and OSD)

授予用户操作RBD图像的权限。当用作Monitor Cap时，它提供了RBD客户端应用程序所需的最小特权；这包括将其他客户端用户列入黑名单的功能。当用作OSD上限时，它为RBD客户端应用程序提供对指定池的读写访问。 Manager上限支持可选的pool和命名空间关键字参数。

#### `profile rbd-mirror` (Monitor only)

授予用户操作RBD图像和检索RBD镜像配置密钥机密的权限。它提供了rbd-mirror守护程序所需的最小特权。

#### `profile rbd-read-only` (Manager and OSD)

向用户授予RBD图像的只读权限。 Manager上限支持可选的pool和命名空间关键字参数。



## Pool

池是用户存储数据的逻辑分区。在Ceph部署中，通常将池创建为类似数据类型的逻辑分区。例如，在将Ceph部署为OpenStack的后端时，典型的部署将具有用于卷，映像，备份和虚拟机以及诸如`client.glance`, `client.cinder` 等用户的池。



## APPLICATION TAGS

Access may be restricted to specific pools as defined by their application metadata. The `*` wildcard may be used for the `key` argument, the `value` argument, or both. `all` is a synony for `*`.



## Namespace

池中的对象可以关联到命名空间-池中对象的逻辑组。用户对池的访问可以与名称空间相关联，这样，用户的读写操作仅在名称空间内进行。写入池中名称空间的对象只能由有权访问该名称空间的用户访问。

> 命名空间主要用于在librados之上编写的应用程序，在此逻辑分组可以减轻创建不同池的需要。 Ceph对象网关对各种元数据对象使用名称空间。

名称空间的基本原理是，出于授权单独的用户集的目的，池可能是一种计算量大的，将数据集分离的方法。例如，每个OSD池应具有约100个放置组。因此，具有1000个OSD的示例群集将为一个池具有100,000个放置组。每个池将在示例性集群中创建另外100,000个放置组。

相比之下，将对象写入名称空间只是将名称空间与对象名称相关联，而没有单独池的计算开销。您可以使用名称空间，而不是为一个用户或一组用户创建单独的池。注意：目前仅可使用librados使用。

使用命名空间功能，访问可能仅限于特定的RADOS命名空间。支持名称空间的有限遍历；如果指定名称空间的最后一个字符是*，则将授予从提供的参数开始的任何名称空间的访问权限。



## 管理用户

用户管理功能使Ceph Storage Cluster管理员能够直接在Ceph Storage Cluster中创建，更新和删除用户。在Ceph存储群集中创建或删除用户时，可能需要将密钥分发给客户端，以便可以将其添加到密钥环中。有关详细信息，请参见[Keyring Management](https://ceph.readthedocs.io/en/latest/rados/operations/user-management/#keyring-management) 。



#### 列出用户

```bash
$ ceph auth ls
```



#### 获取一个用户

```
ceph auth get {TYPE.ID}
```

例如：

```bash
$ ceph auth get client.admin
$ ceph auth get osd.0
```



#### 创建用户

创建用户的几种方法：

- `ceph auth add`: 此命令是添加用户的规范方法。它将创建用户，生成密钥并添加任何指定功能。
- `ceph auth get-or-create`: 该命令通常是创建用户最方便的方法，因为它返回带有用户名（在方括号中）和密钥的密钥文件格式。如果用户已经存在，则此命令仅以密钥文件格式返回用户名和密钥。
- `ceph auth get-or-create-key`: 此命令是创建用户并返回用户密钥（仅）的便捷方法。这对于仅需要密钥的客户端（例如libvirt）很有用。如果用户已经存在，则此命令仅返回密钥。

创建客户端用户时，您可能会创建没有功能的用户。没有能力的用户仅凭身份验证就没有用，因为客户端无法从监视器检索群集映射。但是，如果希望以后再使用ceph auth caps命令推迟添加功能，则可以创建一个没有功能的用户。

典型的用户至少在Ceph监视器上具有读取功能，并且在Ceph OSD上具有读写功能。此外，用户的OSD权限通常仅限于访问特定池：

```bash
$ ceph auth add client.john mon 'allow r' osd 'allow rw pool=liverpool'
$ ceph auth get-or-create client.paul mon 'allow r' osd 'allow rw pool=liverpool'
$ ceph auth get-or-create client.george mon 'allow r' osd 'allow rw pool=liverpool' -o george.keyring
$ ceph auth get-or-create-key client.ringo mon 'allow r' osd 'allow rw pool=liverpool' -o ringo.key
```

> 如果您为用户提供OSD功能，但是您不限制对特定池的访问，则该用户将有权访问群集中的所有池！



#### 修改用户权限

通过`ceph auth caps`命令，您可以指定用户并更改用户的功能。设置新功能将覆盖当前功能。要查看当前功能，请运行 `ceph auth` 获取USERTYPE.USERID。要添加功能，还应该在使用表单时指定现有功能：

```bash
ceph auth caps USERTYPE.USERID {daemon} 'allow [r|w|x|*|...] [pool={pool-name}] [namespace={namespace-name}]' [{daemon} 'allow [r|w|x|*|...] [pool={pool-name}] [namespace={namespace-name}]']
```

例如：

```bash
$ ceph auth get client.john
$ ceph auth caps client.john mon 'allow r' osd 'allow rw pool=liverpool'
$ ceph auth caps client.paul mon 'allow rw' osd 'allow rwx pool=liverpool'
$ ceph auth caps client.brian-manager mon 'allow *' osd 'allow *'
```



#### 删除用户

```bash
$ ceph auth del {TYPE}.{ID}
```



#### 获取用户的 key

```bash
$ ceph auth print-key {TYPE}.{ID}
```



#### 导入用户

keyring 文件中具有用户名，key  和权限。从 keyring 导入用户：

```bash
$ ceph auth import -i /path/to/keyring
```

例如：

```bash
$ sudo ceph auth import -i /etc/ceph/ceph.keyring
```

> ceph存储集群将添加新用户，其密钥和功能，并将更新现有用户，其密钥和功能。



## Keyring 管理

当通过Ceph客户端访问Ceph时，Ceph客户端将查找本地密钥环。Ceph 默认情况下使用以下四个密钥环名称来预设密钥环设置，因此除非您要覆盖默认值（不推荐），否则您不必在Ceph配置文件中进行设置：

- `/etc/ceph/$cluster.$name.keyring`
- `/etc/ceph/$cluster.keyring`
- `/etc/ceph/keyring`
- `/etc/ceph/keyring.bin`

创建用户（例如client.ringo）后，您必须获取密钥并将其添加到 Ceph 客户端上的密钥环中，以便用户可以访问Ceph存储群集。

用户管理部分详细介绍了如何直接在Ceph存储群集中列出，获取，添加，修改和删除用户。但是，Ceph还提供了ceph-authtool实用程序，使您可以从Ceph客户端管理密钥环。



#### 创建 keyring

使用“管理用户”部分中的过程创建用户时，需要向Ceph客户端提供用户密钥，以便Ceph客户端可以检索指定用户的密钥并通过Ceph存储群集进行身份验证。Ceph客户端访问密钥环以查找用户名并检索用户的密钥。

ceph-authtool实用程序允许您创建密钥环。要创建空密钥环，请使用--create-keyring或-C。例如：

```bash
$ ceph-authtool --create-keyring /path/to/keyring
```

当创建具有多个用户的密钥环时，我们建议使用群集名称（例如$ cluster.keyring）作为密钥环文件名，并将其保存在/ etc / ceph目录中，以便密钥环配置默认设置将拾取文件名而不需要 您可以在Ceph配置文件的本地副本中指定它。

```bash
$ sudo ceph-authtool -C /etc/ceph/ceph.keyring
```

当由单个用户创建密钥环时，我们建议使用集群名称，用户类型和用户名，并将其保存在/ etc / ceph目录中。例如，client.admin用户的ceph.client.admin.keyring。

要在/ etc / ceph中创建密钥环，您必须以root用户身份进行。 这意味着该文件将仅对root用户具有rw权限，这在密钥环包含管理员密钥时才适用。 但是，如果您打算为特定用户或一组用户使用密钥环，请确保执行chown或chmod来建立适当的密钥环所有权和访问权。



#### 添加用户到 keyring

如果每个密钥环只希望使用一个用户，则带有-o选项的“获取用户”过程将以密钥环文件格式保存输出。例如，要为client.admin用户创建密钥环，请执行以下操作：

```bash
$ sudo ceph auth get client.admin -o /etc/ceph/ceph.client.admin.keyring
```

请注意，我们为个人用户使用推荐的文件格式。

当您要将用户导入密钥环时，可以使用ceph-authtool来指定目标密钥环和源密钥环。例如：

```bash
$ sudo ceph-authtool /etc/ceph/ceph.keyring --import-keyring /etc/ceph/ceph.client.admin.keyring
```



#### 创建用户

Ceph提供了“添加用户”功能，可以直接在Ceph存储集群中创建用户。但是，您也可以直接在Ceph客户端密钥环上创建用户，密钥和功能。然后，您可以将用户导入Ceph存储集群。例如：

```bash
$ sudo ceph-authtool -n client.ringo --cap osd 'allow rwx' --cap mon 'allow rwx' /etc/ceph/ceph.keyring
```

您还可以创建密钥环，并将新用户同时添加到密钥环。例如：

```bash
$ sudo ceph-authtool -C /etc/ceph/ceph.keyring -n client.ringo --cap osd 'allow rwx' --cap mon 'allow rwx' --gen-key
```

在上述情况下，新用户client.ringo仅在密钥环中。要将新用户添加到Ceph存储群集，您仍然必须将新用户添加到Ceph存储群集。

```bash
$ sudo ceph auth add client.ringo -i /etc/ceph/ceph.keyring
```



#### 修改用户

要修改密钥环中用户记录的功能，请指定密钥环，然后再指定用户。例如：

```bash
$ sudo ceph-authtool /etc/ceph/ceph.keyring -n client.ringo --cap osd 'allow rwx' --cap mon 'allow rwx'
```

要将用户更新到Ceph存储群集，必须将密钥环中的用户更新到Ceph存储群集中的用户条目：

```bash
$ sudo ceph auth import -i /etc/ceph/ceph.keyring
```













































