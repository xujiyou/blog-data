# Dashboard 介绍

官方文档：https://ceph.readthedocs.io/en/latest/mgr/dashboard/

Dashboard 是 ceph-mgr 的一个模块，它的后端代码使用CherryPy框架和自定义REST API。WebUI实现基于Angular / TypeScript。



## 功能概述

Dashboar 的特性如下：

-  多用户和权限管理：Dashboard 支持具有不同权限（角色）的多个用户帐户。 可以在命令行和通过WebUI修改用户帐户和角色。Dashboard 支持多种方法来增强密码安全性，例如 通过执行可配置的密码复杂性规则，强制用户在首次登录后或可配置的时间段后更改密码。 有关详细信息，请参见用户和角色管理。
- 单点登录（SSO）：仪表板支持使用SAML 2.0协议通过外部身份提供商进行身份验证。 有关详细信息，请参阅启用单点登录（SSO）。
- SSL/TLS：Web浏览器和 Dashboard 之间的所有HTTP通信都通过SSL进行保护。 可以使用内置命令来创建自签名证书，但是也可以导入由CA签名和颁发的自定义证书。 有关详细信息，请参见SSL / TLS支持。
- 审计：Dashboard 后端可以配置为在Ceph审核日志中记录所有PUT，POST和DELETE API请求。 有关如何启用此功能的说明，请参见审核API请求。
- 国际化(I18N)：Dashboard 可以在运行时选择的不同的语言。

目前，Dashboard 能够监视和管理Ceph集群的以下方面：

- 集群健康状态：显示总体群集状态，性能和容量指标。
- 嵌入式 Grafana Dashboard：