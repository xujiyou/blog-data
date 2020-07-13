# journalctl 

查看日志所占磁盘容量：

```bash
$ sudo journalctl --disk-usage
```

只显示最新的 n 行

```bash
$ sudo journalctl -n 20
$ journalctl -u cron.service -n 3
```

