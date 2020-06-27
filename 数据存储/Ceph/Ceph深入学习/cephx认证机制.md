---
title: cephx认证机制
date: 2020-06-22 09:42:20
tags:
---

参考：http://www.xuxiaopang.com/2017/08/23/easy-ceph-CephX/

这篇文章挺不错的。

CephX 理解起来很简单，就是整个 Ceph 系统的**用户名/密码**，而这个用户不单单指我们平时在终端敲 `ceph -s` 而生成的 **client**，在这套认证系统中，还有一个特殊的用户群体，那就是 **MON/OSD/MDS**，也就是说，Monitor， OSD， MDS 也都需要一对账号密码来登陆 Ceph 系统。



## CephX 的命名规则

而**用户名/密码**遵循着一定的命名规则：

**用户名**

用户名总体遵循 <**TYPE . ID**> 的命名规则，这里的`TYPE`有三种： `mon`,`osd`,`client`。而 `ID` 根据不同的类型的用户而有所不同：

- `mon` ： `ID` 为**空**。
- `osd` ： `ID` 为 OSD 的 **ID**。
- `client` ： `ID` 为该客户端的名称，比如 `admin`,`cinder`,`nova`。

**密码**

密码通常为包含40个字符的字符串，形如：`AQBh1XlZAAAAABAAcVaBh1p8w4Q3oaGoPW0R8w==`。



## 默认用户

想要和一个 Ceph 集群进行交互，我们通常需要知道最少四条信息，并且是缺一不可的:

- 集群的 fsid。
- 集群的 Monitor 的 IP 地址，必须先连上 MON 之后才能获取集群信息。
- 一个用于登陆的 **用户名**。
- 登陆用户对应的 **密码**。

其实，很多同学会发现，在我们日常和 Ceph 集群交互时，并不需要指定这些参数，就可以执行 `ceph -s` 得到集群的状态。实际上，我们已经使用了 Ceph 提供的几个默认参数，而 `ceph -s` 加上默认参数后的全称是：

```bash
$ ceph -s --conf /etc/ceph/ceph.conf --name client.admin --keyring /etc/ceph/ceph.client.admin.keyring
```

通过下面的命令可以看到密钥文件内容：

```bash
$ ceph auth get client.admin
$ cat /etc/ceph/ceph.client.admin.keyring
```



## 最高权限的用户

最高权限的用户不是 client.admin，而是 mon. ，mon. 的密钥文件在`/var/lib/ceph/mon/ceph-ceph-1/keyring`

如果 client.admin 用户的密钥找不到了，可以通过 mon. 用户找回：

```bash
$ ceph auth get client.admin --name mon. --keyring /var/lib/ceph/mon/ceph-ceph-1/keyring
```



在整个集群启动的时候，首先是 Monitor 启动，再然后是 OSD 启动。在 Monitor 启动的时候，Monitor 会携带自己的秘钥文件启动进程，也就是说，Monitor 启动的时候，是不需要向任何进程进行秘钥认证的，通俗点讲，Monitor 的秘钥哪怕被修改过了，也不会影响 Monitor 的启动。

Monitor 的数据库里面，记录着除了 `mon.` 以外的所有用户密码，在 Monitor 启动之后，才真正开启了认证这个步骤，之后的所有用户想要连接到集群，必须先要通过 fsid 和 MON IP 连上 Ceph 集群，通过了认证之后，就可以正常访问集群了。

OSD 在启动的时候，首先要 `log_to_monitors`，也就是拿着自己的账户密码去登陆集群，这个账户密码在 Monitor 的数据库里有记录，所以如果互相匹配，那么OSD就可以正常启动。

```bash
[root@ceph-1 ~]# cat /var/lib/ceph/osd/ceph-0/keyring
[osd.0]
key = AQDoNe9e8a5EDRAAQ65ZOopNwRJANfcAV+AZzg==
[root@ceph-1 ~]# ceph auth get osd.0
exported keyring for osd.0
[osd.0]
        key = AQDoNe9e8a5EDRAAQ65ZOopNwRJANfcAV+AZzg==
        caps mgr = "allow profile osd"
        caps mon = "allow profile osd"
        caps osd = "allow *"
```





## Caps

仔细查看 `ceph auth list` 的输出，除了用户名和对应的秘钥内容外，还有一个个以 `caps` 开头的内容，这就是 CephX 中对各个用户的权限的细分，比如：读、写、执行等。

而针对不同的应用(mds/mon/osd)，同样的读权限或者写权限的作用是不同的，下面依次对这三个应用的 `r/w/x` 权限进行分析。



### MON

#### r 权限

那么问题来了，想要执行 `ceph -s` 的最低权限是什么呢？ 那就是 `caps mon ="allow r"`，也就是 MON 的 **r** 权限。那么这个读权限到底读了什么呢？首先要强调的一点，这里读的数据都是从 MON 来的，和 OSD 无关。

MON 作为集群的状态维护者，其数据库(`/var/lib/ceph/mon/ceph-$hostname/store.db`)内保存着集群这一系列状态图(Cluster Map)，这些 Map 包含但不限于：

- CRUSH Map
- OSD Map
- MON Map
- MDS Map
- PG Map

而这里的读权限，就是读取这些 Map 的权限，但是这些 Map 的真实内容读取出来没有多大意义，所以以比较友好的指令输出形式展示出来，而这些指令包含但不限于：

```
ceph -s
ceph osd crush dump
ceph osd tree
ceph pg dump 
ceph osd dump
```

只要有了 MON 的 **r** 权限，那么就可以从集群读取所有 MON 维护的 Map 的数据，宏观来看，就是可以读取集群的状态信息(但是不能修改)。

#### w 权限

**w** 权限比较有趣，必须配合 **r** 权限才能有效力，否则，单独 **w** 权限执行指令时，是会一直 `access denied`的。所以我们在测试 **w** 权限时，需要附加上 **r** 权限才行

#### x 权限

MON 的 **x** 权限也比较有趣，因为这个权限仅仅和 `auth` 相关。也就是说，如果你想要执行 `ceph auth list`，`ceph auth get` 之类的所有和 `auth` 相关的指令，那么拥有了 **x** 权限就能执行了。但是和 **w** 权限类似，也需要 **r** 权限组合在一起才能有效力。

#### * 权限

一句话说明 ： `* = rwx`



### OSD

相比于 MON 的各个权限，OSD 的 rw 比较简单理解一下，**r** 权限就是读取对象的权限， **w** 权限就是写对象的权限，**x** 权限比较有趣，可以调用任何 `class-read/class-write`的权限

你可以实现一些自定义的方法，通过调用这些方法，可以读写具有某一类特征的对象，比如都是以`rbd_data`开头的对象。目前只有调用librados层才能使用自定义class方法，而上层RBD，RGW之类的是不能使用的。

***** 权限除了包含了 rwx，还包含了`ceph tell osd.*` 这类admin指令的权限。

官网的这个页面(http://docs.ceph.com/docs/kraken/man/8/ceph-authtool/)比较好的介绍了几个实例，这里简单介绍下一个比较长的指令的意义：

```
osd = "allow class-read object_prefix rbd_children, allow pool templates r class-read, allow pool vms rwx"
```

第一个权限：`object_prefix` 是一个类方法，而这个方法的作用是给予所有以`rbd_children`为名开头的对象的`class-read`也就是读权限。只能读，不能写。

第二个权限：给予池 `templates`的读权限，可以执行class-read方法的权限。也就是说，客户除了可以读这个池的对象外，还能自己实现形如`obejct_prefix`这种系统自带的类方法来读取对象，比如读取具备某一类特征的对象。

第三个权限：给予池 `vms` 的读写以及执行class-read/class-write方法的权限。







