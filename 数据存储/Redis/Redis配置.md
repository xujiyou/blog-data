# Redis 配置

yum 安装的 Redis 配置在 /etc/redis.conf，里面有详细的配置说明，也可以通过下面的方式查看当前系统的所有配置：

```bash
$ redis-cli config get "*"
```

在 6.0.1 中，共有 141 个配置项。

