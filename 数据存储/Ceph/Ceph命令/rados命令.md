---
title: rados命令
date: 2020-06-29 14:35:05
tags:
---

查看帮助信息：

```bash
$ rados -h
```



## POOL COMMANDS

查看 pool 列表：

```bash
$ rados lspools
```

拷贝一个 pool 的内容到另一个 pool

```bash
$ ceph osd pool create test1
$ rados cppool test test1
$ rados -p test1 ls
```

删除 pool 中的全部对象，只会删除对象，而不会删除 pool：

```bash
$ rados purge test1 --yes-i-really-really-mean-it
```

查看使用量：

```bash
$ rados df
```

查看对象列表：

```bash
$ rados -p test1 ls
```



## POOL SNAP COMMANDS

创建一个镜像：

```bash
$ rados mksnap test-snap -p test
```

列出镜像：

```

```















