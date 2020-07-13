# rpm 命令详解

rpm 包删除：

```bash
$ rpm -qa | grep -i httpd-tools
$ rpm -e httpd-tools-2.4.6-90.el7.centos.x86_64
```

查看 rpm 包的依赖：

```bash
$ rpm -qpR percona-xtrabackup-24-2.4.20-1.el7.x86_64.rpm
```

导入 key：

```bash
$ rpm --import /etc/pki/rpm-gpg/RPM*
```

