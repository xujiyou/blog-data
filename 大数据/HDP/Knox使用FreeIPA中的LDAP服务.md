# Knox 使用 FreeIPA 中的 LDAP 服务

在 Ambari 的 Knox 界面上，修改 Knox 的配置。

修改  Advanced admin-topology、 Advanced topology、 Advanced knoxsso-topology中的配置：

```xml
        <param>
            <name>main.ldapRealm.userDnTemplate</name>
            <value>uid=admin,ou=knox,dc=freeipa,dc=testing,dc=com</value>
        </param>
        <param>
            <name>main.ldapRealm.contextFactory.url</name>
            <value>ldap://freeipa.testing.com</value>
        </param>
```

修改完配置，需要在 FreeIPA 的 LDAP 中添加：

```
dn: uid=admin,ou=knox,dc=freeipa,dc=testing,dc=com
cn: Admin
sn: Admin
userPassword: password
```

这样就可以用 LADP 中的用户登录 Knox 了。