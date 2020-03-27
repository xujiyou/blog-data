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



































