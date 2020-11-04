# RabbitMQ 部署

安装：

```bash
$ sudo yum -y install erlang socat
$ erl -version
$ wget https://www.rabbitmq.com/releases/rabbitmq-server/v3.6.10/rabbitmq-server-3.6.10-1.el7.noarch.rpm
$ sudo rpm -Uvh rabbitmq-server-3.6.10-1.el7.noarch.rpm
$ sudo systemctl start rabbitmq-server
$ sudo systemctl enable rabbitmq-server
$ sudo systemctl status rabbitmq-server
```

创建用户并添加所有权限：

```bash
$ sudo rabbitmqctl add_user chris 123
$ sudo rabbitmqctl set_permissions -p / chris . . .
```

