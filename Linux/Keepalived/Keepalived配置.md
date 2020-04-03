# Keepalived 配置

官方配置文档：https://www.keepalived.org/doc/configuration_synopsis.html

Keepalived 共分三块配置，globel，virtual_server 和 vrrp。



# 全局配置

```
global_defs {
    notification_email {
        email
        email
    }
    notification_email_from email
    smtp_server host
    smtp_connect_timeout num
    lvs_id string
}
```

全局配置以 `global_defs` 开头

- `notification_email` 将接受电子邮件的的地址列表
- `notification_email_from` 从哪个账户发出的邮件
- `smtp_server` 邮件服务器地址
- `smtp_connection_timeout` 邮件服务器连接超时时间
- `lvs_id` 指定LVS导向器的名称



## Virtual Server 配置

```
virtual_server (@IP PORT)|(fwmark num) {
    delay_loop num
    lb_algo rr|wrr|lc|wlc|sh|dh|lblc
    lb_kind NAT|DR|TUN
    (nat_mask @IP)
    persistence_timeout num
    persistence_granularity @IP
    virtualhost string
    protocol TCP|UDP

    sorry_server @IP PORT
    real_server @IP PORT {
        weight num
        TCP_CHECK {
            connect_port num
            connect_timeout num
        }
    }
    real_server @IP PORT {
        weight num
        MISC_CHECK {
            misc_path /path_to_script/script.sh
            (or misc_path “ /path_to_script/script.sh <arg_list>”)
        }
    }
}
real_server @IP PORT {
    weight num
    HTTP_GET|SSL_GET {
        url { # You can add multiple url block
            path alphanum
            digest alphanum
        }
        connect_port num
        connect_timeout num
        retry num
        delay_before_retry num
    }
}
```

可以有多个 `virtual_server` 块。

- `fwmark` 指定 virtual server 是 FWMARK 类型的。
- `delay_loop` 指定两次检查之间的间隔，以秒为单位
- `lb_algo` LVS 调度算法
- `lb_kind` LVS集群方式
- `nat_mask` nat子网掩码
- `persistence_timeout` 会话保持时间（秒）
- `persistence_granularity` 为持久连接指定掩码
- `virtualhost` 指定用于HTTP | SSL_GET的HTTP虚拟主机
- `protocol` 指定协议类型，TCP 或 UDP
- `sorry_server` 如果所有真实服务器都已关闭，则将服务器添加到池中
- `real_server` 真是服务器，可以指定多个 real_server 块。
- `weight` 指定实际服务器权重以进行负载平衡决策
- `TCP_CHECK` 使用TCP连接检查实际服务器的可用性
- `MISC_CHECK` 使用用户定义的脚本检查实际服务器的可用性
- `misc_path` 脚本的完整路径
- `HTTP_GET` 使用HTTP GET请求检查实际服务器的可用性
- `SSL_GET` 使用SSL GET请求检查实际服务器的可用性
- `url` 定义 URL
- `path` 指定 URL 路径
- `digest` 指定特定网址路径的摘要
- `connect_port` TCP 连接的端口
- `connect_timeout` TCP 连接超时 时间
- `retry` 重试的最大次数
- `delay_before_retry` 重试之前的延迟。



## VRRP Instance 配置

```
vrrp_sync_group string {
    group {
        string
        string
    }
    notify_master /path_to_script/script_master.sh
        (or notify_master “ /path_to_script/script_master.sh <arg_list>”)
    notify_backup /path_to_script/script_backup.sh
        (or notify_backup “/path_to_script/script_backup.sh <arg_list>”)
    notify_fault /path_to_script/script_fault.sh
        (or notify_fault “ /path_to_script/script_fault.sh <arg_list>”)
}
vrrp_instance string {
    state MASTER|BACKUP
    interface string
    mcast_src_ip @IP
    lvs_sync_daemon_interface string
    virtual_router_id num
    priority num
    advert_int num
    smtp_alert
    authentication {
        auth_type PASS|AH
        auth_pass string
    }
    virtual_ipaddress { # Block limited to 20 IP addresses
        @IP
        @IP
        @IP
    }
    virtual_ipaddress_excluded { # Unlimited IP addresses
        @IP
        @IP
        @IP
    }
    notify_master /path_to_script/script_master.sh
        (or notify_master “ /path_to_script/script_master.sh <arg_list>”)
    notify_backup /path_to_script/script_backup.sh
        (or notify_backup “ /path_to_script/script_backup.sh <arg_list>”)
    notify_fault /path_to_script/script_fault.sh
        (or notify_fault “ /path_to_script/script_fault.sh <arg_list>”)
}
```

- `vrrp_instance` VRRP 的实例
- `state` 指定使用中的实例状态
- `Interface` 网卡，比如 eth0
- `mcast_src_ip` 指定VRRP的src IP地址值advert IP标头
- `lvs_sync_daemon_inteface` 指定要在其上运行LVS sync_daemon的网络接口
- `virtual_router_id` 指定实例所属的VRRP路由器ID
- `priority` 在VRRP路由器中指定实例优先级
- `advert` 指定广告间隔（以秒为单位）（设置为1）
- `smtp_alert` 激活SMTP通知以进行主状态转换
- `authentication` 识别VRRP认证定义块
- `auth_type` 认证类型，PASS 或 AH
- `auth_pass` 指定要使用的密码字符串
- `virtual_ipaddress` 指定 VIP
- `virtual_ipaddress_excluded` 排除在外的 VIP
- `notify_master` 指定在过渡到主状态期间要执行的shell脚本
- `notify_backup` 指定在过渡到备份状态期间要执行的Shell脚本
- `notify_fault` 指定在过渡到故障状态期间要执行的shell脚本
- `vrrp_sync_group` 标识VRRP同步实例组