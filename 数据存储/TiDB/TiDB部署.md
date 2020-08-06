# TiDB 部署

TiDB 是一个 Golang 和 Rust 编写的开源分布式关系型数据库，是一个比较新的技术，从出生起就面向云原生，它的底层是 TiKV，TiKV 是 CNCF 的一个开源项目。

TiDB 采用的是 MySQL 协议，所以使用 MySQL 的程序可以无缝转换。

TiDB 适合高可用、强一致要求较高、数据规模较大等各种应用场景。



## 在 Linux 中部署 TiDB

使用 TiUP 工具来安装 TiDB 集群。下面全部使用一个普通用户执行：

```bash
curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
```

完成后，二进制命令安装在了`/home/bbders/.tiup/bin/tiup`

并且这个脚本还把这个命令在 `~/.bash_profile` 文件中设置到了 PATH 中，使之生效：

```bash
source .bash_profile
```

安装 TiUP cluster 组件：

```
tiup cluster
```

创建 `topology.yaml` 文件，内容如下：

```yaml
global:
  user: "bbders"
  ssh_port: 22
  deploy_dir: "/tidb-deploy"
  data_dir: "/tidb-data"

pd_servers:
  - host: 192.168.112.154
  - host: 192.168.112.155
  - host: 192.168.112.156

tidb_servers:
  - host: 192.168.112.151
  - host: 192.168.112.152
  - host: 192.168.112.153

tikv_servers:
  - host: 192.168.112.151
  - host: 192.168.112.152
  - host: 192.168.112.153

monitoring_servers:
  - host: 192.168.112.156

grafana_servers:
  - host: 192.168.112.156

alertmanager_servers:
  - host: 192.168.112.156
```

创建集群：

```bash
tiup cluster deploy tidb-test v4.0.0 ./topology.yaml
```

检查集群列表：

```bash
tiup cluster list
```

启动集群：

```bash
tiup cluster start tidb-test
```

检查集群的详细情况：

```bash
tiup cluster display tidb-test
```

我的状态如下：

![image-20200806120003188](../../resource/image-20200806120003188.png)



安装 mysql 客户端并登录数据库：

```bash
wget https://repo.mysql.com/mysql57-community-release-el7-11.noarch.rpm
sudo yum localinstall mysql57-community-release-el7-11.noarch.rpm
sudo yum install mysql -y
mysql -u root -h 192.168.112.151 -P 4000
```

随便执行俩操作：

```mysql
mysql> show databases;
mysql> use mysql;
mysql> select user,host from user;
```

















