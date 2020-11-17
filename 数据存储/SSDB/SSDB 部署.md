# SSDB 部署

SSDB 使用源码包进行部署。



## SSDB 安装

SSDB 版本：1.9.4

安装机器：172.19.40.16

安装方式：编译安装

解压源码包，修改其中的 ssdb.conf 的以下内容（密码至少为 32 位）：

```yaml
server:
        ip: 0.0.0.0
        port: 8888
        # auth password must be at least 32 characters
        auth: WaOn13ha@hYaeaXlWaOn13ha@hYaeaXl32
```

编译：

```bash
$ sudo yum install gcc gcc+ gcc-c++ make autoconf -y
$ make
```

安装：

```bash
$ sudo make install
```

结果：

```bash
mkdir -p /usr/local/ssdb
mkdir -p /usr/local/ssdb/_cpy_
mkdir -p /usr/local/ssdb/deps
mkdir -p /usr/local/ssdb/var
mkdir -p /usr/local/ssdb/var_slave
cp -f ssdb-server ssdb.conf ssdb_slave.conf /usr/local/ssdb
cp -rf api /usr/local/ssdb
cp -rf \
        tools/ssdb-bench \
        tools/ssdb-cli tools/ssdb_cli \
        tools/ssdb-cli.cpy tools/ssdb-dump \
        tools/ssdb-repair \
        /usr/local/ssdb
cp -rf deps/cpy /usr/local/ssdb/deps
chmod 755 /usr/local/ssdb
chmod -R ugo+rw /usr/local/ssdb/*
rm -f /usr/local/ssdb/Makefile
```

创建系统用户：

```bash
$ sudo useradd -r ssdb
$ sudo chown -R ssdb:ssdb /usr/local/ssdb
```



编辑 `/usr/lib/systemd/system/ssdb.service`，内容如下：

```
[Unit]
Description=SSDB
Documentation=https://github.com/ideawu/ssdb
Wants=network-online.target
After=network-online.target

[Service]
WorkingDirectory=/usr/local/ssdb
Type=forking

User=ssdb
Group=ssdb

ExecStart=/usr/local/ssdb/ssdb-server -d /usr/local/ssdb/ssdb.conf
ExecStop=/usr/local/ssdb/ssdb-server /usr/local/ssdb/ssdb.conf -s stop
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

启动：

```bash
$ sudo systemctl enable ssdb
$ sudo systemctl start ssdb
```

查看状态：

```bash
$ sudo systemctl status ssdb
$ sudo lsof -i:8888
```



























