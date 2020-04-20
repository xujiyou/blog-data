# Ceph 块储存入门

先创建一个 pool，这里名为 one：

```bash
$ sudo ceph osd pool create one
```

初始化 pool：

```bash
$ sudo rbd pool init one
```

创建 auth：

```bash
$ sudo ceph auth get-or-create client.one mon 'profile rbd' osd 'profile rbd pool=one' mgr 'profile rbd pool=one'
```

查看 auth：

```bash
$ sudo ceph auth get client.one
```



返回：

```
[client.qemu]
        key = AQA0NZ1ebXHfOhAATr93vV5y1maVSXZJEMOWZA==
```

创建一个 10G 的块：

```bash
$ sudo rbd create --size 10240 one/bar
```

查看块列表：

```bash
$ sudo rbd ls one
```

查看块信息：

```bash
$ sudo rbd info one/bar
```

重新设置大小：

```bash
$ sudo rbd resize --size 2048 one/bar --allow-shrink
$ sudo rbd resize --size 20480 one/bar
```

删除块设备：

```bash
$ sudo rbd rm one/bar
```

删除 pool：

```bash
$ sudo ceph osd pool delete one one --yes-i-really-really-mean-it
```

