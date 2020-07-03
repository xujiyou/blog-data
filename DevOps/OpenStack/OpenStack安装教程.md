---
title: OpenStack安装教程
date: 2020-06-24 15:14:32
tags:
---

参考：https://docs.openstack.org/install-guide/

准备三台机器，操作系统为 CentOS 7，hosts 如下：

```
192.168.112.152 test-1
192.168.112.153 test-2
192.168.112.154 test-3
```



## yum 库

当前的最新版本为 **Ussuri** ，但是这个版本还是有坑的，还是用 **Train** 这个版本吧。

这一步在所有节点上安装。

安装 yum 库：

```bash
$ yum install centos-release-openstack-train -y
$ yum install https://rdoproject.org/repos/rdo-release.rpm -y
$ yum upgrade -y
$ yum install python-openstackclient -y #客户端安装
```

我这里关闭了 SELiunx，如果打开了，还需要执行：

```bash
$ yum install openstack-selinux
```



## 组件图

架构图如下，组件挺多的，每个都需要单独安装：

![architecture](../../resource/architecture.png)

全部 service 地址：https://www.openstack.org/software/project-navigator/openstack-components#openstack-services

参考：https://www.cnblogs.com/klb561/p/8660264.html

从左上角开始装。



#### 控制节点架构

控制节点包括以下服务

- 管理支持服务
- 基础管理服务
- 扩展管理服务

管理支持服务包含MySQL与Qpid两个服务

- MySQL：数据库作为基础/扩展服务产生的数据存放的地方

- Qpid：消息代理(也称消息中间件)为其他各种服务之间提供了统一的消息通信服务

基础管理服务包含Keystone，Glance，Nova，Neutron，Horizon五个服务

- Keystone：认证管理服务，提供了其余所有组件的认证信息/令牌的管理，创建，修改等等，使用MySQL作为统一的数据库
- Glance：镜像管理服务，提供了对虚拟机部署的时候所能提供的镜像的管理，包含镜像的导入，格式，以及制作相应的模板
- Nova：计算管理服务，提供了对计算节点的Nova的管理，使用Nova-API进行通信
- Neutron：网络管理服务，提供了对网络节点的网络拓扑管理，同时提供Neutron在Horizon的管理面板
- Horizon：控制台服务，提供了以Web的形式对所有节点的所有服务的管理，通常把该服务称为DashBoard

扩展管理服务包含Cinder，Swift，Trove，Heat，Centimeter五个服务

- Cinder：提供管理存储节点的Cinder相关，同时提供Cinder在Horizon中的管理面板
- Swift：提供管理存储节点的Swift相关，同时提供Swift在Horizon中的管理面板
- Trove：提供管理数据库节点的Trove相关，同时提供Trove在Horizon中的管理面板
- Heat：提供了基于模板来实现云环境中资源的初始化，依赖关系处理，部署等基本操作，也可以解决自动收缩,负载均衡等高级特性。
- Centimeter：提供对物理资源以及虚拟资源的监控，并记录这些数据，对该数据进行分析，在一定条件下触发相应动作

控制节点一般来说只需要一个网络端口用于通信/管理各个节点。



#### 网络节点架构

网络节点仅包含Neutron服务

- Neutron：负责管理私有网段与公有网段的通信，以及管理虚拟机网络之间的通信/拓扑，管理虚拟机之上的防火等等



#### 计算节点架构

计算节点包含Nova，Neutron，Telemeter三个服务

基础服务

- Nova：提供虚拟机的创建，运行，迁移，快照等各种围绕虚拟机的服务，并提供API与控制节点对接，由控制节点下发任务

- Neutron：提供计算节点与网络节点之间的通信服务

扩展服务

- Telmeter：提供计算节点的监控代理，将虚拟机的情况反馈给控制节点，是Centimeter的代理服务



#### 存储节点架构

存储节点包含Cinder，Swift等服务

- Cinder：块存储服务，提供相应的块存储，简单来说，就是虚拟出一块磁盘，可以挂载到相应的虚拟机之上，不受文件系统等因素影响，对虚拟机来说，这个操作就像是新加了一块硬盘，可以完成对磁盘的任何操作，包括挂载，卸载，格式化，转换文件系统等等操作，大多应用于虚拟机空间不足的情况下的空间扩容等等

- Swift：对象存储服务，提供相应的对象存储，简单来说，就是虚拟出一块磁盘空间，可以在这个空间当中存放文件，也仅仅只能存放文件，不能进行格式化，转换文件系统，大多应用于云磁盘/文件



## 架构图

参考：https://docs.openstack.org/install-guide/get-started-conceptual-architecture.html

![OpenStack conceptual architecture](../../resource/openstack_kilo_conceptual_arch.png)

参考：https://docs.openstack.org/install-guide/get-started-logical-architecture.html

![Logical architecture](../../resource/openstack-arch-kilo-logical-v1.png)

对小白来说，巨复杂。。。



## 准备

一共就三台机器，每台机器都是 24 核心 CPU、32 G内存、两块 200 G 的盘。

准备三台机器上安装 Ceph、一台机器当作控制节点，一台当作计算节点。

下面先安装一些基础服务，包括数据库、消息中间件、Etcd 等，再安装 OpenStack 的一些 Service。



## 安装基础服务

参考：https://docs.openstack.org/install-guide/environment.html

下面的组件全部在 test-1 上安装和设置。

#### 密码设置

要提前设置一些密码，后续都通过环境变量的方式提供，生成随机密码可以使用以下命令：

```bash
$ openssl rand -hex 10
```

在 `/etc/profile` 中设置环境变量：

```bash
export ADMIN_PASS=fc05e1929b2c057a4098
export CINDER_DBPASS=BBDERS1@bbdops.com
export CINDER_PASS=fc05e1929b2c057a4098
export DASH_DBPASS=fc05e1929b2c057a4098
export DEMO_PASS=fc05e1929b2c057a4098
export GLANCE_DBPASS=BBDERS1@bbdops.com
export GLANCE_PASS=fc05e1929b2c057a4098
export KEYSTONE_DBPASS=BBDERS1@bbdops.com
export METADATA_SECRET=fc05e1929b2c057a4098
export NEUTRON_DBPASS=BBDERS1@bbdops.com
export NEUTRON_PASS=fc05e1929b2c057a4098
export NOVA_DBPASS=BBDERS1@bbdops.com
export NOVA_PASS=fc05e1929b2c057a4098
export PLACEMENT_PASS=fc05e1929b2c057a4098
export RABBIT_PASS=fc05e1929b2c057a4098
```

使之生效：

```bash
$ source /etc/profile
```



#### 安装 MySQL

安装一个单节点的 mysql，在 test-1 上安装，安装教程： [MySQL最新版本安装.md](../../数据存储/MySQL/MySQL最新版本安装.md) 

额外添加一个参数，调大最大连接数：

```
max_connections=300
```

查看最大连接数：

```mysql
mysql> show variables like '%max_connections%';
```

查看当前连接数：

```mysql
mysql> show status like 'Threads%';
```

Threads_connected 是当前连接数，Threads_running 是并发数。





#### 安装 RabbitMQ

RabbitMQ 是一个由 Erlang 语言开发的 AMQP 的开源实现。是一个老牌的消息中间件。

```bash
$ yum install rabbitmq-server
```

启动 RabbitMQ：

```bash
$ systemctl enable rabbitmq-server.service
$ systemctl start rabbitmq-server.service
```

添加用户：

```bash
$ rabbitmqctl add_user openstack $RABBIT_PASS
```

为 openstack 用户添加配置、读、写权限：

````bash
$ rabbitmqctl set_permissions openstack ".*" ".*" ".*"
````



#### 安装 Memcached

*memcached*是一套分布式的高速缓存系統。安装：

```bash
$ yum install memcached python-memcached
```

修改 `/etc/sysconfig/memcached` 配置文件，以允许外部访问，将 `OPTIONS="-l 127.0.0.1,::1"` 改为 ：

```
OPTIONS=""
```

启动：

```bash
$ systemctl enable memcached.service
$ systemctl start memcached.service
```



#### 安装 etcd

```bash
$ yum install etcd
```

修改配置文件 `/etc/etcd/etcd.conf` 如下：

```properties
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="http://192.168.112.152:2380"
ETCD_LISTEN_CLIENT_URLS="http://192.168.112.152:2379"
ETCD_NAME="controller"

ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.112.152:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.112.152:2379"
ETCD_INITIAL_CLUSTER="controller=http://192.168.112.152:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster-01"
ETCD_INITIAL_CLUSTER_STATE="new"
```

启动：

```bash
$ systemctl enable etcd
$ systemctl start etcd
```



## 安装 OpenStack Service

最小安装参考：https://docs.openstack.org/install-guide/openstack-services.html#minimal-deployment-for-train

需要安装 Keystone、Glance、Placement、Nova、Neutron 和 Horizon、Cinder



## 安装 Keystone

在 test-1 上安装 Keystone

创建 mysql 用户及库（mysql 8.0）：

```mysql
mysql> CREATE DATABASE keystone;
mysql> CREATE USER keystone IDENTIFIED BY 'BBDERS1@bbdops.com';
mysql> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%';
mysql> FLUSH PRIVILEGES;
```

在 test-1 上安装 keystone ：

```bash
$ yum install openstack-keystone httpd mod_wsgi
```

修改配置文件 `/etc/keystone/keystone.conf` 如下：

```toml
[database]
connection = mysql+pymysql://keystone:BBDERS1%40bbdops.com@test-1/keystone

[token]
provider = fernet
```

由于我设置的密码中有特殊字符，所以需要urlencode，@ 进行 urlencode 之后就是 %40 。

填充服务数据库：

```bash
$ su -s /bin/sh -c "keystone-manage db_sync" keystone
```

初始化Fernet密钥存储库:

```
$ keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
$ keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
```

启动 keystone 服务：

```bash
$ keystone-manage bootstrap --bootstrap-password $ADMIN_PASS \
  --bootstrap-admin-url http://test-1:5000/v3/ \
  --bootstrap-internal-url http://test-1:5000/v3/ \
  --bootstrap-public-url http://test-1:5000/v3/ \
  --bootstrap-region-id RegionOne
```

配置 httpd 服务器，修改 `/etc/httpd/conf/httpd.conf` 配置文件：

```
ServerName test-1
```

创建软连接：

```bash
$ ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
```

启动 httpd 服务：

```bash
$ systemctl enable httpd.service
$ systemctl start httpd.service
```

在 /etc/profile 中配置环境变量，在三台机器上都要配置：

```bash
export OS_USERNAME=admin
export OS_PASSWORD=$ADMIN_PASS
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://test-1:5000/v3
```

使之生效：

```bash
$ source /etc/profile
```





---



创建默认 domain：

```bash
$ openstack domain create --description "An Example Domain" example
```

创建 service 项目：

```bash
$ openstack project create --domain default --description "Service Project" service
```

创建`myproject` project :

```bash
$ openstack project create --domain default --description "Demo Project" myproject
```

创建 **myuser** 用户：

```bash
$ openstack user create --domain default --password-prompt myuser
```

需要设置密码，我这里设置的 123456

创建 **myrole** 权限：

```bash
$ openstack role create myrole
```

把 myrole 权限加入到 myproject 和 myuser 中：

```bash
$ openstack role add --project myproject --user myuser myrole
```



---



验证：

```bash
$ unset OS_AUTH_URL OS_PASSWORD
$ openstack --os-auth-url http://controller:5000/v3 \
  --os-project-domain-name Default --os-user-domain-name Default \
  --os-project-name admin --os-username admin token issue
```

输入 admin 用户的密码。

再验证 myproject：

```bash
$ openstack --os-auth-url http://test-1:5000/v3 \
  --os-project-domain-name Default --os-user-domain-name Default \
  --os-project-name myproject --os-username myuser token issue
```

输入密码，我这里是 123456。



---



创建 **admin-openrc** 文件：

```bash
export OS_USERNAME=admin
export OS_PASSWORD=$ADMIN_PASS
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://test-1:5000/v3
export OS_IDENTITY_API_VERSION=3
```

创建 **demo-openrc** 文件：

```bash
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=myproject
export OS_USERNAME=myuser
export OS_PASSWORD=123456
export OS_AUTH_URL=http://test-1:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```

使之生效：

```bash
$ . admin-openrc
```

请求认证token：

```bash
$ openstack token issue
```



## 安装 Glance

Glance 是镜像服务。在 test-1 上安装 Glance。

创建 mysql 用户及库：

```mysql
mysql> CREATE DATABASE glance;
mysql> CREATE USER glance IDENTIFIED BY 'BBDERS1@bbdops.com';
mysql> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%';
mysql> FLUSH PRIVILEGES;
```

创建 **glance** 用户：

```bash
$ openstack user create --domain default --password-prompt glance
```

密码为上面的 GLANCE_PASS ，即 fc05e1929b2c057a4098

为 glance 用户添加 admin 权限：

```bash
$ openstack role add --project service --user glance admin
```

创建 glance service ：

```bash
$ openstack service create --name glance --description "OpenStack Image" image
```

创建 Image service API endpoints:

```bash
$ openstack endpoint create --region RegionOne image public http://test-1:9292
$ openstack endpoint create --region RegionOne image internal http://test-1:9292
$ openstack endpoint create --region RegionOne image admin http://test-1:9292
```

安装 glance 组件：

```bash
$ yum install openstack-glance
```

修改 `/etc/glance/glance-api.conf` 文件：

```toml
[database]
connection = mysql+pymysql://glance:BBDERS1%40bbdops.com@test-1/glance

[keystone_authtoken]
www_authenticate_uri  = http://test-1:5000
auth_url = http://test-1:5000
memcached_servers = test-1:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = fc05e1929b2c057a4098

[paste_deploy]
flavor = keystone

[glance_store]
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/
```

初始化数据库：

```bash
$ su -s /bin/sh -c "glance-manage db_sync" glance
```

启动 Glance 服务：

```bash
$ systemctl enable openstack-glance-api.service
$ systemctl start openstack-glance-api.service
```



## 安装 Placement

在 test-1 上安装 Placement

创建 mysql 库和用户：

```mysql
mysql> CREATE DATABASE placement;
mysql> CREATE USER placement IDENTIFIED BY 'BBDERS1@bbdops.com';
mysql> GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'%';
mysql> FLUSH PRIVILEGES;
```

创建用户：

```bash
$ openstack user create --domain default --password-prompt placement
```

密码是上面的 PLACEMENT_PASS ，即 fc05e1929b2c057a4098

添加 placement service ：

```bash
$ openstack role add --project service --user placement admin
```

创建Placement API entry：

```bash
$ openstack service create --name placement --description "Placement API" placement
```

创建 Placement API service endpoints :

```bash
$ openstack endpoint create --region RegionOne placement public http://test-1:8778
$ openstack endpoint create --region RegionOne placement internal http://test-1:8778
$ openstack endpoint create --region RegionOne placement admin http://test-1:8778
```

安装 Placement 组件：

```bash
$ yum install openstack-placement-api
```

修改 `/etc/placement/placement.conf` 文件：

```toml
[placement_database]
connection = mysql+pymysql://placement:BBDERS1%40bbdops.com@test-1/placement

[api]
auth_strategy = keystone

[keystone_authtoken]
auth_url = http://test-1:5000/v3
memcached_servers = test-1:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = placement
password = fc05e1929b2c057a4098
```

初始化数据库：

```bash
$ su -s /bin/sh -c "placement-manage db sync" placement
```

重启 httpd 服务：

```bash
$ systemctl restart httpd
```

验证：

```bash
$ placement-status upgrade check
$ pip install osc-placement
$ openstack --os-placement-api-version 1.2 resource class list --sort-column name
$ openstack --os-placement-api-version 1.6 trait list --sort-column name
```

这里出现一个错误，如果不解决，后续也有问题，错误：

```
Expecting value: line 1 column 1 (char 0)
```

解决方案，在 `/etc/httpd/conf.d/00-placement-api.conf` 中的 `<VirtualHost *:8778>` 内部加入以下代码：

```xml
  <Directory /usr/bin>
    <IfVersion >= 2.4>
      Require all granted
    </IfVersion>
    <IfVersion < 2.4>
      Order allow,deny
      Allow from all
    </IfVersion>
  </Directory>
```

重启 httpd：

```bash
$ systemctl restart httpd
```

再次验证：

```bash
$ openstack --os-placement-api-version 1.2 resource class list --sort-column name
$ openstack --os-placement-api-version 1.6 trait list --sort-column name
```





## 安装 Nova

需要先安装 Nova 控制节点，再安装 Nova 计算节点



#### 安装 Nova 控制节点

在 test-1 上安装 Nova 控制节点。

创建 mysql 用户和库：

````mysql
mysql> CREATE DATABASE nova_api;
mysql> CREATE DATABASE nova;
mysql> CREATE DATABASE nova_cell0;
mysql> CREATE USER nova IDENTIFIED BY 'BBDERS1@bbdops.com';
mysql> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%';
mysql> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%';
mysql> GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%';
mysql> FLUSH PRIVILEGES;
````

创建 nova 用户：

```bash
$ openstack user create --domain default --password-prompt nova
```

密码是 NOVA_PASS ，即 fc05e1929b2c057a4098 。

为 nova 添加 admin 权限：

```bash
$ openstack role add --project service --user nova admin
```

创建 `nova` service entity：

```bash
openstack service create --name nova --description "OpenStack Compute" compute
```

创建 Compute API service endpoints：

```bash
$ openstack endpoint create --region RegionOne compute public http://test-1:8774/v2.1
$ openstack endpoint create --region RegionOne compute internal http://test-1:8774/v2.1
$ openstack endpoint create --region RegionOne compute admin http://test-1:8774/v2.1
```

安装 nova：

```bash
$ yum install openstack-nova-api openstack-nova-conductor openstack-nova-novncproxy openstack-nova-scheduler
```

修改 `/etc/nova/nova.conf` 文件：

```toml
[DEFAULT]
enabled_apis=osapi_compute,metadata
block_device_allocate_retries=300
block_device_allocate_retries_interval=3


[api_database]
connection = mysql+pymysql://nova:BBDERS1%40bbdops.com@test-1/nova_api

[database]
connection = mysql+pymysql://nova:BBDERS1%40bbdops.com@test-1/nova

[DEFAULT]
transport_url=rabbit://openstack:fc05e1929b2c057a4098@test-1:5672/

[api]
auth_strategy = keystone

[keystone_authtoken]
www_authenticate_uri = http://test-1:5000/
auth_url = http://test-1:5000/
memcached_servers = test-1:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = nova
password = fc05e1929b2c057a4098

[DEFAULT]
my_ip=192.168.112.152

[DEFAULT]
use_neutron = true
firewall_driver = nova.virt.firewall.NoopFirewallDriver

[vnc]
enabled = true
server_listen = $my_ip
server_proxyclient_address = $my_ip

[glance]
api_servers=http://test-1:9292

[oslo_concurrency]
lock_path = /var/lib/nova/tmp

[placement]
region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://test-1:5000/v3
username = placement
password = fc05e1929b2c057a4098
```

初始化数据库：

```bash
$ su -s /bin/sh -c "nova-manage api_db sync" nova
```

注册 **cell0** 数据库：

```bash
$ su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
```

注册 cell1 数据库：

```bash
$ su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
```

填充 nove 数据库：

```bash
$ su -s /bin/sh -c "nova-manage db sync" nova
```

验证 cell0 和 cell1 是否被注册了：

```bash
$ su -s /bin/sh -c "nova-manage cell_v2 list_cells" nova
```

启动 nova：

```bash
$ systemctl enable \
    openstack-nova-api.service \
    openstack-nova-scheduler.service \
    openstack-nova-conductor.service \
    openstack-nova-novncproxy.service
$ systemctl start \
    openstack-nova-api.service \
    openstack-nova-scheduler.service \
    openstack-nova-conductor.service \
    openstack-nova-novncproxy.service
```

检查更新：

```bash
$ nova-status upgrade check
```





#### 安装 Nova 计算节点

在 test-2 和 test-3 上安装：

```bash
$ yum install openstack-nova-compute
```

修改 `/etc/nova/nova.conf` 文件：

```toml
[DEFAULT]
enabled_apis = osapi_compute,metadata
block_device_allocate_retries=300
block_device_allocate_retries_interval=3

[DEFAULT]
transport_url=rabbit://openstack:fc05e1929b2c057a4098@test-1

[api]
auth_strategy = keystone

[keystone_authtoken]
www_authenticate_uri = http://test-1:5000/
auth_url = http://test-1:5000/
memcached_servers = test-1:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = nova
password = fc05e1929b2c057a4098

[DEFAULT]
my_ip=192.168.112.154

[DEFAULT]
use_neutron = true
firewall_driver = nova.virt.firewall.NoopFirewallDriver

[vnc]
enabled = true
server_listen = 0.0.0.0
server_proxyclient_address = $my_ip
novncproxy_base_url = http://test-1:6080/vnc_auto.html

[glance]
api_servers=http://test-1:9292

[oslo_concurrency]
lock_path = /var/lib/nova/tmp

[placement]
region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://test-1:5000/v3
username = placement
password = fc05e1929b2c057a4098
```

执行：

```bash
$ egrep -c '(vmx|svm)' /proc/cpuinfo
```

如果返回了 0 ，还需要配置：

```toml
[libvirt]
virt_type=qemu
```

启动 Nova 计算节点：

```bash
$ systemctl enable libvirtd.service openstack-nova-compute.service
$ systemctl start libvirtd.service openstack-nova-compute.service
```

查看有哪些计算节点：

```bash
$ openstack compute service list --service nova-compute
```

输入密码：fc05e1929b2c057a4098

发现计算节点：

```bash
$ su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova
```

每次加入新节点后，都要执行 `nova-manage cell_v2 discover_hosts` 命令。



----



#### 验证 Nova 安装

```bash
$ openstack catalog list
$ openstack catalog list
$ openstack image list
$ nova-status upgrade check
```





## 安装 Neutron

Neutron 提供网络服务。Neutron 也分控制节点和计算节点。



#### 安装 Neutron 控制节点

创建 mysql 库和用户：

```mysql
mysql> CREATE DATABASE neutron;
mysql> CREATE USER neutron IDENTIFIED BY 'BBDERS1@bbdops.com';
mysql> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%';
mysql> FLUSH PRIVILEGES;
```

创建 **neutron** 用户：

```bash
$ openstack user create --domain default --password-prompt neutron
```

使用 NEUTRON_PASS 为密码，即 fc05e1929b2c057a4098。

为 neutron 用户添加 admin 权限：

```bash
$ openstack role add --project service --user neutron admin
```

创建 `neutron` service entity：

```bash
$ openstack service create --name neutron --description "OpenStack Networking" network
```

创建 Networking service API endpoints：

```bash
$ openstack endpoint create --region RegionOne network public http://test-1:9696
$ openstack endpoint create --region RegionOne network internal http://test-1:9696
$ openstack endpoint create --region RegionOne network admin http://test-1:9696
```



---



这里配置网络有两种选项，下面来部署比较简单的 选项一：

```bash
$ yum install openstack-neutron openstack-neutron-ml2 openstack-neutron-linuxbridge ebtables
```

修改 `/etc/neutron/neutron.conf` 文件：

```toml
[database]
connection = mysql+pymysql://neutron:BBDERS1%40bbdops.com@test-1/neutron

[DEFAULT]
core_plugin = ml2
service_plugins =

[DEFAULT]
transport_url=rabbit://openstack:fc05e1929b2c057a4098@test-1

[DEFAULT]
auth_strategy = keystone

[keystone_authtoken]
www_authenticate_uri = http://test-1:5000
auth_url = http://test-1:5000
memcached_servers = test-1:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = fc05e1929b2c057a4098

[DEFAULT]
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true

[nova]
auth_url = http://test-1:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = fc05e1929b2c057a4098

[oslo_concurrency]
lock_path = /var/lib/neutron/tmp
```

修改 `/etc/neutron/plugins/ml2/ml2_conf.ini` 文件，添加以下内容：

```toml
[ml2]
type_drivers = flat,vlan
tenant_network_types =
mechanism_drivers = linuxbridge
extension_drivers = port_security

[ml2_type_flat]
flat_networks = provider

[securitygroup]
enable_ipset = true
```

修改 `/etc/neutron/plugins/ml2/linuxbridge_agent.ini` 文件，添加以下内容：

```toml
[linux_bridge]
physical_interface_mappings = provider:ens192

[vxlan]
enable_vxlan = false

[securitygroup]
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```

修改 `/etc/neutron/dhcp_agent.ini` 文件，加入以下配置：

```toml
interface_driver = linuxbridge
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = true
```



---



修改 `/etc/neutron/metadata_agent.ini` 文件，加入以下内容：

```toml
[DEFAULT]
nova_metadata_host = controller
metadata_proxy_shared_secret = fc05e1929b2c057a4098
```

修改 `/etc/nova/nova.conf` ，加入以下内容：

````
[neutron]
auth_url = http://test-1:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = fc05e1929b2c057a4098
service_metadata_proxy = true
metadata_proxy_shared_secret = fc05e1929b2c057a4098
````

启动 Neutron：

````bash
$ ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
$ su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
  --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
$ systemctl restart openstack-nova-api.service
$ systemctl enable neutron-server.service \
  neutron-linuxbridge-agent.service neutron-dhcp-agent.service \
  neutron-metadata-agent.service
$ systemctl start neutron-server.service \
  neutron-linuxbridge-agent.service neutron-dhcp-agent.service \
  neutron-metadata-agent.service
````

创建网络：

```bash
$ openstack network create  --share --external \
  --provider-physical-network provider \
  --provider-network-type flat provider
```

创建子网：

```bash
$ openstack subnet create --network provider \
  --allocation-pool start=192.168.112.155,end=192.168.112.160 \
  --dns-nameserver 10.28.100.100 --gateway 192.168.112.1 \
  --subnet-range 192.168.112.0/24 provider
```

结果如下：

![image-20200630175243693](../../resource/image-20200630175243693.png)



----



#### 安装 Neutron 计算节点

在 test-2 和 test-3 上安装 Neutron 计算节点。

```bash
$ yum install openstack-neutron-linuxbridge ebtables ipset
```

修改 `/etc/neutron/neutron.conf` 文件:

```toml
[DEFAULT]
transport_url=rabbit://openstack:fc05e1929b2c057a4098@test-1

[DEFAULT]
auth_strategy = keystone

[keystone_authtoken]
www_authenticate_uri = http://test-1:5000
auth_url = http://test-1:5000
memcached_servers = test-1:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = fc05e1929b2c057a4098

[oslo_concurrency]
lock_path = /var/lib/neutron/tmp
```

修改 `/etc/neutron/plugins/ml2/linuxbridge_agent.ini` 文件：

```toml
[linux_bridge]
physical_interface_mappings = provider:ens192

[vxlan]
enable_vxlan = false

[securitygroup]
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```

修改 `/etc/nova/nova.conf` 文件：

```toml
[neutron]
url = http://test-1:9696
auth_url = http://test-1:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = fc05e1929b2c057a4098
```

重启计算服务：

```bash
$ systemctl restart openstack-nova-compute.service
```

启动网络计算服务：

```bash
$ systemctl enable neutron-linuxbridge-agent.service
$ systemctl start neutron-linuxbridge-agent.service
```

验证：

```bash
$ openstack extension list --network
```

输入密码：fc05e1929b2c057a4098

查看网络节点列表：

```bash
$ openstack network agent list
```



## 安装 CInder

Cinder 分为 控制节点、储存节点、备份节点



#### 安装 Cinder 控制节点

添加 mysql 库和 用户：

```mysql
mysql> CREATE DATABASE cinder;
mysql> CREATE USER cinder IDENTIFIED BY 'BBDERS1@bbdops.com';
mysql> GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'%';
mysql> FLUSH PRIVILEGES;
```

创建 cinder 用户：

```bash
$ openstack user create --domain default --password-prompt cinder
```

密码是 CINDER_PASS，即 fc05e1929b2c057a4098。

为 cinder 用户绑定 admin 权限：

```bash
$ openstack role add --project service --user cinder admin
```

创建 `cinderv2` and `cinderv3` service entities：

```bash
$ openstack service create --name cinderv2 \
  --description "OpenStack Block Storage" volumev2
$ openstack service create --name cinderv3 \
  --description "OpenStack Block Storage" volumev3
```

创建 Block Storage service API endpoints：

```bash
$ openstack endpoint create --region RegionOne \
  volumev2 public http://test-1:8776/v2/%\(project_id\)s
$ openstack endpoint create --region RegionOne \
  volumev2 internal http://test-1:8776/v2/%\(project_id\)s
$ openstack endpoint create --region RegionOne \
  volumev2 admin http://test-1:8776/v2/%\(project_id\)s
  
$ openstack endpoint create --region RegionOne \
  volumev3 public http://test-1:8776/v3/%\(project_id\)s
$ openstack endpoint create --region RegionOne \
  volumev3 internal http://test-1:8776/v3/%\(project_id\)s
$ openstack endpoint create --region RegionOne \
  volumev3 admin http://test-1:8776/v3/%\(project_id\)s
```

安装 cinder：

```bash
$ yum install openstack-cinder
```

修改 `/etc/cinder/cinder.conf` 文件：

```toml
[database]
connection = mysql+pymysql://cinder:BBDERS1%40bbdops.com@test-1/cinder

[DEFAULT]
transport_url=rabbit://openstack:fc05e1929b2c057a4098@test-1:5672/
auth_strategy = keystone
my_ip = 192.168.112.152

[keystone_authtoken]
www_authenticate_uri = http://test-1:5000
auth_url = http://test-1:5000
memcached_servers = test-1:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = cinder
password = fc05e1929b2c057a4098

[oslo_concurrency]
lock_path = /var/lib/cinder/tmp
```

初始化数据库：

```bash
$ su -s /bin/sh -c "cinder-manage db sync" cinder
```

配置计算节点使用块储存，在 全部节点 上修改 `/etc/nova/nova.conf` :

```
[cinder]
os_region_name = RegionOne
```

重启 nova-api :

```bash
$ systemctl restart openstack-nova-api.service
```

启动块储存控制节点的服务：

```bash
$ systemctl enable openstack-cinder-api.service openstack-cinder-scheduler.service
$ systemctl start openstack-cinder-api.service openstack-cinder-scheduler.service
```



#### 安装 Cinder 储存节点

在 test-2 中安装

```bash
$ yum install lvm2 device-mapper-persistent-data
```

启动 lvm：

```bash
$ systemctl enable lvm2-lvmetad.service
$ systemctl start lvm2-lvmetad.service
```

准备几块磁盘，创建pv：

```bash
$ pvcreate /dev/sdb
$ pvcreate /dev/sdc
```

创建 vg：

```bash
$ vgcreate cinder-volumes /dev/sdb /dev/sdc
```

在 `/etc/lvm/lvm.conf` 中 的 devices 块中添加：

```
filter = [ "a/sdb/", "a/sdc/" "r/.*/"]
```

安装 Cinder 储存组件：

```bash
$ yum install openstack-cinder targetcli python-keystone
```

修改 `/etc/cinder/cinder.conf` ：

```toml
[database]
connection = mysql+pymysql://cinder:BBDERS1%40bbdops.com@test-1/cinder

[DEFAULT]
transport_url=rabbit://openstack:fc05e1929b2c057a4098@test-1
auth_strategy = keystone
my_ip = 192.168.112.153
enabled_backends = lvm
glance_api_servers = http://test-1:9292

[keystone_authtoken]
www_authenticate_uri = http://test-1:5000
auth_url = http://test-1:5000
memcached_servers = test-1:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = cinder
password = fc05e1929b2c057a4098

[lvm]
volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
volume_group = cinder-volumes
target_protocol = iscsi
target_helper = lioadm

[oslo_concurrency]
lock_path = /var/lib/cinder/tmp
```

启动 Cinder 储存节点：

```bash
$ systemctl enable openstack-cinder-volume.service target.service
$ systemctl start openstack-cinder-volume.service target.service
```

验证：

```bash
$ openstack volume service list
```

输入密码 fc05e1929b2c057a4098





## 安装 Horizon

Horizon就是 Openstack 的 Dashboard。

在 test-1 上安装：

```bash
$ yum install openstack-dashboard
```

修改 `/etc/openstack-dashboard/local_settings` 文件：

```
OPENSTACK_HOST = "test-1"
ALLOWED_HOSTS = ['*']
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
CACHES = {
    'default': {
         'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
         'LOCATION': 'test-1:11211',
    }
}

TIME_ZONE = "Asia/Shanghai"

OPENSTACK_NEUTRON_NETWORK = {
    'enable_auto_allocated_network': False,
    'enable_distributed_router': False,
    'enable_fip_topology_check': True,
    'enable_ha_router': False,
    'enable_ipv6': True,
    # TODO(amotoki): Drop OPENSTACK_NEUTRON_NETWORK completely from here.
    # enable_quotas has the different default value here.
    'enable_quotas': False,
    'enable_rbac_policy': True,
    'enable_router': True,
    'enable_lb': False,
    'enable_firewall': False,
    'enable_vpn': False,

    'default_dns_nameservers': [],
    'supported_provider_types': ['*'],
    'segmentation_id_range': {},
    'extra_provider_types': {},
    'supported_vnic_types': ['*'],
    'physical_networks': [],

}


OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True
OPENSTACK_API_VERSIONS = {
    "identity": 3,
    "image": 2,
    "volume": 3,
}
OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = "Default"
OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"

WEBROOT = "/dashboard/"
```



在 `/etc/httpd/conf.d/openstack-dashboard.conf` 中添加：

```
WSGIApplicationGroup %{GLOBAL}
```

重启 httpd 和 缓存服务：

```bash
$ systemctl restart httpd.service memcached.service
```

测试访问：http://test-1/dashboard

域填写 default，用户名为 admin,密码为  fc05e1929b2c057a4098

在界面上创建一个镜像，镜像需要特殊定制的。然后查看镜像列表：

```bash
$ glance image-list
```

然后在 管理员 -> 实例类型中创建一个实例类型。

最后在 项目 -> 计算 -> 实例中创建实例。

我这里可以创建完成！



## 错误



#### 错误1 

在创建实例时，报错说卷创建错误：

```
Volume 0e4150db-567 f-4ae0-a947-8fc7a0d624f0 did not finish being created even after we waited 150 seconds or 61 attempts. And its status is downloading.
```

解决方法：在 nova 的控制和计算节点的 `/etc/nova/nova.conf` 中添加以下配置：

```
block_device_allocate_retries=300
block_device_allocate_retries_interval=3
```

然后重启 Nova 相关的服务，在控制节点：

```bash
$ systemctl restart \
      openstack-nova-api.service \
      openstack-nova-scheduler.service \
      openstack-nova-conductor.service \
      openstack-nova-novncproxy.service
```

在计算节点：

```bash
$ systemctl restart libvirtd.service openstack-nova-compute.service
```



## 总结

跟着官方的教程走，可以手动安装一个集群，中间会有一两个小错误，在网上都可以找到解决方案。

















