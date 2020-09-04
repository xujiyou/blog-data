# IP 转发

一次用 Dokcer，明明启用了端口转发，防火墙也没开，可是外部就是不能访问，然后打开 IP 转发就可以了：

```bash
$ echo 1 > /proc/sys/net/ipv4/ip_forward
$ echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
$ sysctl -p
```

