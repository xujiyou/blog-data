# DNS 服务器修改记录

参见：https://wiki2.xbits.net:4430/linux:dns:unbound_nsd?s%5B%5D=nsd

[NSD](https://www.nlnetlabs.nl/projects/nsd/about/)是一个高性能的`Authoritative ONLY` DNS服务器。



## 添加 DNS 记录

主服务器  `10.28.70.14` 从服务器 `10.28.70.13`  vip `10.28.70.100`

首先，修改记录，并修改 serial。修改 serial 的原因是让从服务器备份主服务器的配置。

```bash
$ vi /etc/nsd/zones/bbdops.com.zone
```

修改 serial 内容如下：

```
                                        2020091502      ; serial
```

这里，前面是当前日期，如果一天内修改了多次，后面的 01 要递增，比如是第二次修改，这里的 01 要变成 02。

添加的规则如下：

```
hdp1.prod      IN      A       10.28.63.11
hdp2.prod      IN      A       10.28.63.12
hdp3.prod      IN      A       10.28.63.13
hdp4.prod      IN      A       10.28.63.14
hdp5.prod      IN      A       10.28.63.15
kubenode1.prod IN      A       10.28.63.16
kubenode2.prod IN      A       10.28.63.19
kubenode3.prod IN      A       10.28.63.20
```

配置文件修改完成后，执行以下命令使之生效：

```bash
$ nsd-checkzone bbdops.com /etc/nsd/zones/bbdops.com.zone # 检测zone和对应zone文件的语法
$ nsd-control reload bbdops.com # 没问题的话就reload
$ unbound-control reload # unbound 也 reload 以下，加载新的缓存
```

测试：

```bash
$ ping hdp1.prod.bbdops.com
```

















