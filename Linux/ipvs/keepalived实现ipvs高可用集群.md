---
title: keepalived实现ipvs高可用集群
date: 2020-07-26 17:07:49
tags:
---

有 6 台机器，hosts 如下：

```
192.168.112.151 test-1
192.168.112.152 test-2
192.168.112.153 test-3
192.168.112.154 test-4
192.168.112.156 test-5
192.168.112.157 test-6
```

分配一个 VIP 为 192.168.112.160

test-1 和 test-2 之间通过 keepalived 实现高可用的 ipvs 集群。后端 test-3 和 test-3 运行 HTTP 服务。



## test-1 与 test-2 配置

安装工具：

```bash
$ yum install keepalived ipvsadm  -y
```

keepalived 配置 VRRP 实例，test-1 为 master，test-2 为 backup。

test-1 的`/etc/keepalived/keepalived.conf` 文件内容如下 :

```
global_defs {
   notification_email {
      root@localhost
   }
   notification_email_from bbders@localhost
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
   vrrp_skip_check_adv_addr
   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

vrrp_instance VI_1 {
    state MASTER
    interface ens192
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.112.160
    }
}
```

test-2 的这个文件只是把 MASTER 替换为 BACKUP。

在 test-1 和 test-2 上启动 keepalived：

```bash
$ systemctl start keepalived
```

这时在 test-1 上使用 `ip a` 会在 ens192 网卡上看到两个 ipv4 地址，其中一个就是 VIP，如果把 test-1 的 keepalived 停掉，会发现这个 VIP 飘到了 test-2 上。



以上我们实现通过 keepalived 实现了 ip 漂移，接下来我们可以通过 keepalived 配置 lvs 调度。当客户端访问虚拟 ip 192.168.112.160时，我们可以把客户端的请求调度到后端的RS1和RS2去。

在 test-1和 test-2 的 `/etc/keepalived/keepalived.conf` 文件内，添加 virtual_server 模块：

```
virtual_server 192.168.112.160 80 {
        delay_loop 3
        lb_algo rr
        lb_kind DR
        protocol TCP

        real_server 192.168.112.153 80 {
                weight 1
                HTTP_GET {
                    url {
                        path /
                        status_code 200
                    }
                    connect_timeout 1
                    nb_get_retry 3
                    delay_before_retry 1
                }
        }
        real_server 192.168.112.154 80 {
                weight 1
                HTTP_GET {
                    url {
                        path /
                        status_code 200
                    }
                    connect_timeout 1
                    nb_get_retry 3
                    delay_before_retry 1
                }
        }
}
```

重启 keepalived：

```bash
$ systemctl restart keepalived
```

查看 ipvs 记录：

```bash
$ ipvsadm -Ln
$ ipvsadm-save -n
-A -t 192.168.112.160:80 -s rr
-a -t 192.168.112.160:80 -r 192.168.112.153:80 -g -w 1
-a -t 192.168.112.160:80 -r 192.168.112.154:80 -g -w 1
```





## test-3 和 test-4 服务

```bash
$ cd /tmp/
$ echo "test-3" >> index.html
$ python -m SimpleHTTPServer 80

$ cd /tmp/
$ echo "test-4" >> index.html
$ python -m SimpleHTTPServer 80
```



## 错误

在具体操作时，发现了两个错误。



#### VIP ping 不通

发现 VIP ping 不通，发送 VIP 的 Mac 地址：

```bash
$ arping -I ens192 -c 10 -s 192.168.112.160 192.168.112.156
```

发送之后，发现 arping 可以通，但是 ping 不通。

最后在 https://segmentfault.com/q/1010000009646984 找到了答案：在 global_defs 中不要配置 vrrp_strict。

vrrp_strict 的意思是严格执行VRRP协议规范，此模式不支持节点单播。

从这个地方可以看出，VIP 与 ARP 协议是有渊源的。



#### 服务访问不通

VIP 可以 ping 通了，但是访问不通，是因为我在第一次设置 virtual_server 时，把 lb_kind 设置为了 DR，即直接路由，后面我改成 NAT，重启 keepalived 就可以了。

参考：https://blog.csdn.net/charthyf/article/details/81456872



## 最终配置

所以最终的 master 的配置如下：

```
global_defs {
   notification_email {
      root@localhost
   }
   notification_email_from bbders@localhost
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
   vrrp_skip_check_adv_addr
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

vrrp_instance VI_1 {
    state MASTER
    interface ens192
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.112.160
    }
}

virtual_server 192.168.112.160 80 {
        delay_loop 3
        lb_algo rr
        lb_kind NAT
        protocol TCP

        real_server 192.168.112.153 80 {
                weight 1
                HTTP_GET {
                    url {
                        path /
                        status_code 200
                    }
                    connect_timeout 1
                    nb_get_retry 3
                    delay_before_retry 1
                }
        }
        real_server 192.168.112.154 80 {
                weight 1
                HTTP_GET {
                    url {
                        path /
                        status_code 200
                    }
                    connect_timeout 1
                    nb_get_retry 3
                    delay_before_retry 1
                }
        }
}
```





## 测试

```bash
$ ping 192.168.112.160
$ curl http://192.168.112.160
```

连续访问，会发现交替返回 test-3 和 test-4，说明配置生效了。



## 总结

从上边的实际操作，结合  [ipvs使用实例.md](ipvs使用实例.md)  这篇文章，可以发现，LVS 或者说 IPVS，只能实现负载均衡的功能，无法实现高可用和健康检查。比如实现 IPVS 规则的那台机器挂了，服务也就完全挂了；再比如说某一台 real server 挂了，IPVS 还是一如既往的往这台已经挂的 real server 上分配流量。

Keepalived 在 IPVS 的基础上，增加了 VIP 和健康检查，实现了高可用，并且在异常时，也可以发邮件，及时通知运维人员修复。



参考：https://tuonioooo-notebook.gitbook.io/high-concurent-load/lvs/lvsshi-zhan











