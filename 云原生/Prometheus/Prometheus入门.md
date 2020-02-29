# Prometheus 入门

官方文档：https://prometheus.io/docs/introduction/overview/

## 二进制运行

首先去 https://prometheus.io/download/ 下载对应系统的二进制包。

下载，解压，进入目录

查看帮助：

```bash
$ ./prometheus --help
```

prometheus.yml 是配置文件。

启动：

```bash
$ ./prometheus --config.file=prometheus.yml
```



