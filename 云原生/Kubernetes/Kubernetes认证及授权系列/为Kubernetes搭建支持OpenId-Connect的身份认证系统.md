# 为 Kubernetes 搭建支持 OpenId Connect 的身份认证系统

博文：https://www.ibm.com/developerworks/cn/cloud/library/cl-lo-openid-connect-kubernetes-authentication/index.html

博文2:https://www.ibm.com/developerworks/cn/cloud/library/cl-lo-openid-connect-kubernetes-authentication2/index.html

除了 ServiceAccount（用于 Pod 内） 外，k8s 几乎不提供任何用户身份管理和认证的功能。kube-apiserver 认识的用户名其实是证书中的 CN 字段，认识的用户组是证书中的 O 字段。

本文介绍如何在 Kubernetes 中使用 OpenID Connect（OIDC）Token 进行身份认证。kube-apiserver 中有 `oidc` 相关的配置。

OpenID Connect（OIDC）是一个协议，官方网站：https://openid.net/connect/



## 一个基于 Keycloak 的验证系统

为进一步详细说明 Kubernetes OIDC 认证流程，我们来亲手搭建一套验证系统。

首先需要搭建一个 Auth Server，它是用来提供用户身份标识，也就是 id_token 的。Kubernetes 的语境下，我们也叫它 Identity Provider，简称 IdP。Kubernetes 对 IdP 有三个要求：

1. 支持 [OpenID Connection Discovery](https://openid.net/specs/openid-connect-discovery-1_0.html)。并不是所有的 IdP 都支持。
2. 支持 TLS 通讯。Kubernetes (Api Server) 要求和 IdP 的通讯只能使用 TLS，并且只能使用 Kubernetes 支持的加密方式。
3. IdP 拥有一个 CA 签署的 Certificate（即便是您使用一个非商业 CA——比如您自己组织的私有 CA， 或者使用自签名的 Certificate）。

Keycloak 是 OIDC 的一个实现。

官方文档：https://www.keycloak.org/ 

Github：https://github.com/keycloak/keycloak/



### 安装并运行 Keycloak

下载地址：https://www.keycloak.org/downloads.html

下面 Server 压缩包，解压。

先单机启动，修改 `standalone/configuration/standalone.xml` ，将 `jboss.bind.address` 中的地址修改为 `0..0.0.0` ，以允许外网访问，然后添加用户：

```bash
$ ./bin/add-user-keycloak.sh -u admin
```

输入密码之后就创建了一个用户，下面启动服务：

```bash
$ ./bin/standalone.sh > keycloak.log 2>&1 &
```

然后浏览器打开 `http://localhost:8080` 

下一步在页面上创建 Realm ，教程：https://www.keycloak.org/docs/latest/getting_started/index.html

在创建 k8s 集群时已经创建了 CA 及其私钥，这里就不生成了，具体教程查看： [Kubernetes证书生成.md](Kubernetes证书生成.md) 



## 生成 keystore

Keycloak 是 Java based 的工程，它只接受 Java keystore (jks) 格式的秘钥对。我们可以用 JDK 中的 keytool 来生成 Keycloak 接受的 keystore，并把我们之前生成的 Private Key 和 Certificate 导入其中。这里需要注意的一点是，JDK keytool 并不支持直接导入 keypair，我们需要使用 PKCS12 格式做一个转换中介。具体生成 keystore 的脚本如下：

```bash
$ openssl pkcs12 -export -out keycloak.p12 -inkey keycloak-key.pem -in keycloak.pem -certfile /etc/kubernetes/cert/ca.pem
password:passw0rd
$ keytool -importkeystore -deststorepass 'passw0rd' -destkeystore keycloak.jks -srckeystore keycloak.p12 -srcstoretype PKCS12
```

检查 jks 文件：

```bash
$ keytool -list -keystore keycloak.jks -v
```

密码输入passw0rd



### 配置 Keycloak

修改 `standalone/configuration/standalone.xml`  来使用上一步生成的 keycloak.jks。修改其中的：

```xml
             <security-realm name="ApplicationRealm">
                <server-identities>
                    <ssl>
                        <keystore path="keycloak.jks" relative-to="jboss.server.config.dir" keystore-password="passw0rd" alias="1" key-password="passw0rd" generate-self-signed-certificate-host="localhost"/>
                    </ssl>
                </server-identities>
                <authentication>
                    <local default-user="$local" allowed-users="*" skip-group-loading="true"/>
                    <properties path="application-users.properties" relative-to="jboss.server.config.dir"/>
                </authentication>
                <authorization>
                    <properties path="application-roles.properties" relative-to="jboss.server.config.dir"/>
                </authorization>
            </security-realm>
```



## systemd 方式安装

上面的安装方式可以测试用，生产环境还是要 systemd 部署。

教程：https://medium.com/@hasnat.saeed/setup-keycloak-server-on-ubuntu-18-04-ed8c7c79a2d9



## 使用

按照 https://www.ibm.com/developerworks/cn/cloud/library/cl-lo-openid-connect-kubernetes-authentication2/index.html 这个教程基本就可以跑起来。这里记录一下错误和命令：

首先要依次创建 realm, client, user。user 需要激活，重新登录一下，记住登录的是 自己创建的 realm，而不是 master realm。

然后配置 kube-apiserver ，这里记录下我的 配置：

```
--oidc-client-id=kubernetes \
--oidc-issuer-url=https://fueltank-1:8443/auth/realms/kubernetes \
--oidc-ca-file=/etc/kubernetes/cert/ca.pem \
--oidc-username-claim=preferred_username \
--oidc-username-prefix=- \
--oidc-groups-claim=groups \
```

获取各种 token：

```bash
$ curl -k 'https://fueltank-1:8443/auth/realms/kubernetes/protocol/openid-connect/token' -d "client_id=kubernetes" -d "client_secret=7b032277-aee5-438b-810c-863891e5b091" -d "response_type=code token" -d "grant_type=password" -d "username=theone" -d "password=123456" -d "scope=openid" | json_reformat
```

解码 token 的线上地址：https://jwt.io/

kubectl 客户端配置：

```bash
$ kubectl config set-cluster fueltank --certificate-authority=/home/admin/k8s-cluster/cert/ca.pem --embed-certs=true --server=https://fueltank-1:6443

$ kubectl config set-credentials theone --auth-provider=oidc --auth-provider-arg=idp-issuer-url=https://fueltank-1:8443/auth/realms/kubernetes --auth-provider-arg=client-id=kubernetes --auth-provider-arg=client-secret=7b032277-aee5-438b-810c-863891e5b091 --auth-provider-arg=refresh-token=eyJhbGciOiJIUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICI0OTIyZGRhYy1jZTEzLTQ3NWQtOTNkMi0zMjJhMmQ4YzFiMTkifQ.eyJleHAiOjE1ODU0NjY4MjUsImlhdCI6MTU4NTQ2NTAyNSwianRpIjoiNjQxNGU2ZDYtNGE4Yy00ZjBiLTkwNTUtYmJiZTlhZTFhZTA3IiwiaXNzIjoiaHR0cHM6Ly9mdWVsdGFuay0xOjg0NDMvYXV0aC9yZWFsbXMva3ViZXJuZXRlcyIsImF1ZCI6Imh0dHBzOi8vZnVlbHRhbmstMTo4NDQzL2F1dGgvcmVhbG1zL2t1YmVybmV0ZXMiLCJzdWIiOiIzMTY0NjU4Ni1hZWFmLTQ0N2MtOGMzYS04NzE2OTc3NTFlYmIiLCJ0eXAiOiJSZWZyZXNoIiwiYXpwIjoia3ViZXJuZXRlcyIsInNlc3Npb25fc3RhdGUiOiI1NTc4NDU0MC0zNjJhLTRhNWYtOWYxNC1kZjZmODQ4MDJjMjIiLCJzY29wZSI6Im9wZW5pZCBlbWFpbCBwcm9maWxlIn0.dLG52S_Dsz2bZP8Ebhx67XVwFvN298D-5pK5iWl83Ak --auth-provider-arg=idp-certificate-authority=/etc/kubernetes/cert/ca.pem --auth-provider-arg=id-token=eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJrWE9HQmFZa2hpT3pqVW9obzc5ekV5OVhVamVtYm1XblpCdDlaTGlDSndJIn0.eyJleHAiOjE1ODU0NjUzMjUsImlhdCI6MTU4NTQ2NTAyNSwiYXV0aF90aW1lIjowLCJqdGkiOiJjYjA5MjEwOC00MzcxLTQ4MWMtYjE3YS02N2FhY2MyYzQ1ZGEiLCJpc3MiOiJodHRwczovL2Z1ZWx0YW5rLTE6ODQ0My9hdXRoL3JlYWxtcy9rdWJlcm5ldGVzIiwiYXVkIjoia3ViZXJuZXRlcyIsInN1YiI6IjMxNjQ2NTg2LWFlYWYtNDQ3Yy04YzNhLTg3MTY5Nzc1MWViYiIsInR5cCI6IklEIiwiYXpwIjoia3ViZXJuZXRlcyIsInNlc3Npb25fc3RhdGUiOiI1NTc4NDU0MC0zNjJhLTRhNWYtOWYxNC1kZjZmODQ4MDJjMjIiLCJhY3IiOiIxIiwiZW1haWxfdmVyaWZpZWQiOmZhbHNlLCJwcmVmZXJyZWRfdXNlcm5hbWUiOiJ0aGVvbmUifQ.i1_-HLtTu2fMHsC-xvd-qtKJp2aUv6J8UtlFUR8oU5e_iLn6EtlK2LHk1TfNFH91xuQyaHAm07_SQWR2TpYEKedy6h-SCPLKrPmklMfls6p3J8_4s240PkUzR2jHoIUWf38qgefU8YymXs8vcwMpM_sdwK0Ju1YEf_FFVzR4A--ORvxTtFH3lQReUqVgXCHWf5x83kszzFRs4jh-xbYiX_l1UDNH5Bwxo3i1IvBLwFdP14bh9Z2c2Dfh1IZUJsCWC6JqZEOVYu_59dLc5S52ZdVbotngq3I4ysVhul6gJJCDnfr_0vQflY2152dANUVi_h9GkullvfSxC3Zvp_SVFw

$ kubectl config set-context theone@kubernetes --cluster=kubernetes --user=theone --namespace=default

$ kubectl config use-context theone@kubernetes
```



设置 refresh-token 的过期时间在：https://fueltank-1:8443/auth/admin/master/console/#/realms/kubernetes/token-settings

其中，SSO Session Idle 和 SSO Session Max 是 refresh-token 的过期时间。Access Token Lifespan 是 id-token 的过期时间。



最后，如果想给用户加分组可以在 keycloak 中添加用户属性和 Mapper。





























