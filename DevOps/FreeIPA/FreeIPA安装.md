# FreeIPA 安装

参考：https://www.sysit.cn/blog/post/sysit/FreeIPA-HA%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE

`FreeIPA`建立在著名的开源组件和标准协议之上，是一个集成的安全信息管理解决方案，具有易于管理、安装和配置任务自动化的特点。它整合了`389-ds（LDAP）`、`Kerberos`、`NTP`、`bind`、`apache httpd`、`tomcat`核心软件包，形成一个以`389-ds（LDAP）`为数据存储后端，`Kerberos`为验证前端，`bind`为主机识别，并且具有统一的命令行管理工具及`apache+tomcat`提供的`web`管理界面的集成信息管理系统。



## 安装

在 CentOS8 上安装 FreeIPA

```bash
$ sudo yum -y install @idm:DL1
$ sudo yum install ipa-server bind bind-dyndb-ldap ipa-server-dns 
```

配置：

```bash
$ sudo ipa-server-install --setup-dns --forwarder=10.10.10.10
```

过程如下：

```
The log file for this installation can be found in /var/log/ipaserver-install.log
==============================================================================
This program will set up the IPA Server.

This includes:
  * Configure a stand-alone CA (dogtag) for certificate management
  * Configure the Network Time Daemon (ntpd)
  * Create and configure an instance of Directory Server
  * Create and configure a Kerberos Key Distribution Center (KDC)
  * Configure Apache (httpd)
  * Configure DNS (bind)
  * Configure the KDC to enable PKINIT

To accept the default shown in brackets, press the Enter key.

Enter the fully qualified domain name of the computer
on which you're setting up server software. Using the form
<hostname>.<domainname>
Example: master.example.com.


Server host name [freeipa-1.hdp.testing.com]: 

Warning: skipping DNS resolution of host freeipa-1.hdp.testing.com
The domain name has been determined based on the host name.

Please confirm the domain name [hdp.testing.com]: 

The kerberos protocol requires a Realm name to be defined.
This is typically the domain name converted to uppercase.

Please provide a realm name [HDP.TESTING.COM]: 
Certain directory server operations require an administrative user.
This user is referred to as the Directory Manager and has full access
to the Directory for system management tasks and will be added to the
instance of directory server created for IPA.
The password must be at least 8 characters long.

Directory Manager password: 
Password (confirm): 

The IPA server requires an administrative user, named 'admin'.
This user is a regular system account used for IPA server administration.

IPA admin password: 
Password (confirm): 

Checking DNS domain hdp.testing.com., please wait ...
WARNING: No network interface matches the IP address 10.10.10.29
Checking DNS forwarders, please wait ...
DNS server 10.10.10.10: answer to query '. SOA' is missing DNSSEC signatures (no RRSIG data)
Please fix forwarder configuration to enable DNSSEC support.
(For BIND 9 add directive "dnssec-enable yes;" to "options {}")
WARNING: DNSSEC validation will be disabled
Do you want to search for missing reverse zones? [yes]: yes
Do you want to create reverse zone for IP 10.28.109.29 [yes]: yes
Please specify the reverse zone name [10.10.10.in-addr.arpa.]: 
Using reverse zone(s) 10.10.10.in-addr.arpa.

The IPA Master Server will be configured with:
Hostname:       freeipa-1.hdp.testing.com
IP address(es): 10.28.109.29
Domain name:    hdp.testing.com
Realm name:     HDP.TESTING.COM

BIND DNS server will be configured to serve IPA domain with:
Forwarders:       10.10.10.10
Forward policy:   only
Reverse zone(s):  10.10.10.in-addr.arpa.

Continue to configure the system with these values? [no]: yes

The following operations may take some minutes to complete.
Please wait until the prompt is returned.

Configuring NTP daemon (ntpd)
  [1/4]: stopping ntpd
  [2/4]: writing configuration
  [3/4]: configuring ntpd to start on boot
  [4/4]: starting ntpd
Done configuring NTP daemon (ntpd).
Configuring directory server (dirsrv). Estimated time: 30 seconds
  [1/45]: creating directory server instance
  [2/45]: enabling ldapi
  [3/45]: configure autobind for root
  [4/45]: stopping directory server
  [5/45]: updating configuration in dse.ldif
  [6/45]: starting directory server
  [7/45]: adding default schema
  [8/45]: enabling memberof plugin
  [9/45]: enabling winsync plugin
  [10/45]: configure password logging
  [11/45]: configuring replication version plugin
  [12/45]: enabling IPA enrollment plugin
  [13/45]: configuring uniqueness plugin
  [14/45]: configuring uuid plugin
  [15/45]: configuring modrdn plugin
  [16/45]: configuring DNS plugin
  [17/45]: enabling entryUSN plugin
  [18/45]: configuring lockout plugin
  [19/45]: configuring topology plugin
  [20/45]: creating indices
  [21/45]: enabling referential integrity plugin
  [22/45]: configuring certmap.conf
  [23/45]: configure new location for managed entries
  [24/45]: configure dirsrv ccache
  [25/45]: enabling SASL mapping fallback
  [26/45]: restarting directory server
  [27/45]: adding sasl mappings to the directory
  [28/45]: adding default layout
  [29/45]: adding delegation layout
  [30/45]: creating container for managed entries
  [31/45]: configuring user private groups
  [32/45]: configuring netgroups from hostgroups
  [33/45]: creating default Sudo bind user
  [34/45]: creating default Auto Member layout
  [35/45]: adding range check plugin
  [36/45]: creating default HBAC rule allow_all
  [37/45]: adding entries for topology management
  [38/45]: initializing group membership
  [39/45]: adding master entry
  [40/45]: initializing domain level
  [41/45]: configuring Posix uid/gid generation
  [42/45]: adding replication acis
  [43/45]: activating sidgen plugin
  [44/45]: activating extdom plugin
  [45/45]: configuring directory to start on boot
Done configuring directory server (dirsrv).
Configuring Kerberos KDC (krb5kdc)
  [1/10]: adding kerberos container to the directory
  [2/10]: configuring KDC
  [3/10]: initialize kerberos container
  [4/10]: adding default ACIs
  [5/10]: creating a keytab for the directory
  [6/10]: creating a keytab for the machine
  [7/10]: adding the password extension to the directory
  [8/10]: creating anonymous principal
  [9/10]: starting the KDC
  [10/10]: configuring KDC to start on boot
Done configuring Kerberos KDC (krb5kdc).
Configuring kadmin
  [1/2]: starting kadmin 
  [2/2]: configuring kadmin to start on boot
Done configuring kadmin.
Configuring ipa-custodia
  [1/5]: Making sure custodia container exists
  [2/5]: Generating ipa-custodia config file
  [3/5]: Generating ipa-custodia keys
  [4/5]: starting ipa-custodia 
  [5/5]: configuring ipa-custodia to start on boot
Done configuring ipa-custodia.
Configuring certificate server (pki-tomcatd). Estimated time: 3 minutes
  [1/29]: configuring certificate server instance
  [2/29]: reindex attributes
  [3/29]: exporting Dogtag certificate store pin
  [4/29]: stopping certificate server instance to update CS.cfg
  [5/29]: backing up CS.cfg
  [6/29]: disabling nonces
  [7/29]: set up CRL publishing
  [8/29]: enable PKIX certificate path discovery and validation
  [9/29]: starting certificate server instance
  [10/29]: configure certmonger for renewals
  [11/29]: requesting RA certificate from CA
  [12/29]: setting audit signing renewal to 2 years
  [13/29]: restarting certificate server
  [14/29]: publishing the CA certificate
  [15/29]: adding RA agent as a trusted user
  [16/29]: authorizing RA to modify profiles
  [17/29]: authorizing RA to manage lightweight CAs
  [18/29]: Ensure lightweight CAs container exists
  [19/29]: configure certificate renewals
  [20/29]: configure Server-Cert certificate renewal
  [21/29]: Configure HTTP to proxy connections
  [22/29]: restarting certificate server
  [23/29]: updating IPA configuration
  [24/29]: enabling CA instance
  [25/29]: migrating certificate profiles to LDAP
  [26/29]: importing IPA certificate profiles
  [27/29]: adding default CA ACL
  [28/29]: adding 'ipa' CA entry
  [29/29]: configuring certmonger renewal for lightweight CAs
Done configuring certificate server (pki-tomcatd).
Configuring directory server (dirsrv)
  [1/3]: configuring TLS for DS instance
  [2/3]: adding CA certificate entry
  [3/3]: restarting directory server
Done configuring directory server (dirsrv).
Configuring ipa-otpd
  [1/2]: starting ipa-otpd 
  [2/2]: configuring ipa-otpd to start on boot
Done configuring ipa-otpd.
Configuring the web interface (httpd)
  [1/22]: stopping httpd
  [2/22]: setting mod_nss port to 443
  [3/22]: setting mod_nss cipher suite
  [4/22]: setting mod_nss protocol list to TLSv1.2
  [5/22]: setting mod_nss password file
  [6/22]: enabling mod_nss renegotiate
  [7/22]: disabling mod_nss OCSP
  [8/22]: adding URL rewriting rules
  [9/22]: configuring httpd
  [10/22]: setting up httpd keytab
  [11/22]: configuring Gssproxy
  [12/22]: setting up ssl
  [13/22]: configure certmonger for renewals
  [14/22]: importing CA certificates from LDAP
  [15/22]: publish CA cert
  [16/22]: clean up any existing httpd ccaches
  [17/22]: configuring SELinux for httpd
  [18/22]: create KDC proxy config
  [19/22]: enable KDC proxy
  [20/22]: starting httpd
  [21/22]: configuring httpd to start on boot
  [22/22]: enabling oddjobd
Done configuring the web interface (httpd).
Configuring Kerberos KDC (krb5kdc)
  [1/1]: installing X509 Certificate for PKINIT
Done configuring Kerberos KDC (krb5kdc).
Applying LDAP updates
Upgrading IPA:. Estimated time: 1 minute 30 seconds
  [1/10]: stopping directory server
  [2/10]: saving configuration
  [3/10]: disabling listeners
  [4/10]: enabling DS global lock
  [5/10]: disabling Schema Compat
  [6/10]: starting directory server
  [7/10]: upgrading server
  [8/10]: stopping directory server
  [9/10]: restoring configuration
  [10/10]: starting directory server
Done.
Restarting the KDC
Configuring DNS (named)
  [1/12]: generating rndc key file
  [2/12]: adding DNS container
  [3/12]: setting up our zone
  [4/12]: setting up reverse zone
  [5/12]: setting up our own record
  [6/12]: setting up records for other masters
  [7/12]: adding NS record to the zones
  [8/12]: setting up kerberos principal
  [9/12]: setting up named.conf
  [10/12]: setting up server configuration
  [11/12]: configuring named to start on boot
  [12/12]: changing resolv.conf to point to ourselves
Done configuring DNS (named).
Restarting the web server to pick up resolv.conf changes
Configuring DNS key synchronization service (ipa-dnskeysyncd)
  [1/7]: checking status
  [2/7]: setting up bind-dyndb-ldap working directory
  [3/7]: setting up kerberos principal
  [4/7]: setting up SoftHSM
  [5/7]: adding DNSSEC containers
  [6/7]: creating replica keys
  [7/7]: configuring ipa-dnskeysyncd to start on boot
Done configuring DNS key synchronization service (ipa-dnskeysyncd).
Restarting ipa-dnskeysyncd
Restarting named
Updating DNS system records
Configuring client side components
Using existing certificate '/etc/ipa/ca.crt'.
Client hostname: freeipa-1.hdp.testing.com
Realm: HDP.TESTING.COM
DNS Domain: hdp.testing.com
IPA Server: freeipa-1.hdp.testing.com
BaseDN: dc=hdp,dc=testing,dc=com

Skipping synchronizing time with NTP server.
New SSSD config will be created
Configured sudoers in /etc/nsswitch.conf
Configured /etc/sssd/sssd.conf
trying https://freeipa-1.hdp.testing.com/ipa/json
[try 1]: Forwarding 'schema' to json server 'https://freeipa-1.hdp.testing.com/ipa/json'
trying https://freeipa-1.hdp.testing.com/ipa/session/json
[try 1]: Forwarding 'ping' to json server 'https://freeipa-1.hdp.testing.com/ipa/session/json'
[try 1]: Forwarding 'ca_is_enabled' to json server 'https://freeipa-1.hdp.testing.com/ipa/session/json'
Systemwide CA database updated.
Adding SSH public key from /etc/ssh/ssh_host_rsa_key.pub
Adding SSH public key from /etc/ssh/ssh_host_ecdsa_key.pub
Adding SSH public key from /etc/ssh/ssh_host_ed25519_key.pub
[try 1]: Forwarding 'host_mod' to json server 'https://freeipa-1.hdp.testing.com/ipa/session/json'
SSSD enabled
Configured /etc/openldap/ldap.conf
Configured /etc/ssh/ssh_config
Configured /etc/ssh/sshd_config
Configuring hdp.testing.com as NIS domain.
Client configuration complete.
The ipa-client-install command was successful

==============================================================================
Setup complete

Next steps:
        1. You must make sure these network ports are open:
                TCP Ports:
                  * 80, 443: HTTP/HTTPS
                  * 389, 636: LDAP/LDAPS
                  * 88, 464: kerberos
                  * 53: bind
                UDP Ports:
                  * 88, 464: kerberos
                  * 53: bind
                  * 123: ntp

        2. You can now obtain a kerberos ticket using the command: 'kinit admin'
           This ticket will allow you to use the IPA tools (e.g., ipa user-add)
           and the web user interface.

Be sure to back up the CA certificates stored in /root/cacert.p12
These files are required to create replicas. The password for these
files is the Directory Manager password
```

示例密码：admin@password

查看用户信息，需要先认证：

```bash
$ sudo kinit admin
Password for admin@testing.com
$ sudo ipa user-find --all
```

查看状态：

```bash
$ sudo ipactl status
Directory Service: RUNNING
krb5kdc Service: RUNNING
kadmin Service: RUNNING
httpd Service: RUNNING
ipa-custodia Service: RUNNING
pki-tomcatd Service: RUNNING
ipa-otpd Service: RUNNING
ipa: INFO: The ipactl command was successful
```

浏览器打开：http://node1.testing.com



## 安装replica

在 freeipa-1 上执行：

```bash
$ sudo kinit admin
$ sudo ipa host-add --force --ip-address=10.10.10.28 freeipa-2.hdp.testing.com
$ sudo ipa host-find
$ sudo ipa host-find |grep "Host name"
```

查看当前所有 Zone：

```bash
$ sudo ipa dnszone-find | grep "Zone name" 
```

设置`dnszone` 的 `allow-sync-ptr` 属性 ( 注意：先执行以下命令，再去执行添加或删除机器操作 )

```bash
$ sudo ipa dnszone-mod hdp.testing.com. --allow-sync-ptr=true
$ sudo ipa dnszone-mod 10.10.10.in-addr.arpa. --allow-sync-ptr=true
```



在 freeipa-2 修改 `/etc/resolv.conf`

```
search hdp.testing.com
nameserver 10.28.109.29
```

```bash
$ sudo ipa-client-install --force-join
```

在 freeipa-2 上安装 freeipa ：

```bash
$ sudo kinit admin
$ sudo klist
$ sudo ipa-replica-install --setup-dns --forwarder 10.10.10.29
```



在 freeipa-1 上查看是否成功：

```bash
$ sudo ipa-replica-manage list
$ sudo ipa host-find
```

> 上述执行完成之后，执行`ipactl status`检查发现`ipa2` 比 `ipa1` 少了`pki-tomcatd`这组件。执行ipa-ca-install安装即可
>
> 在 freeipa-2 上执行：
>
> ```bash
> $ sudo ipa-ca-install
> ```



## 用户管理

FreeIPA 可以管理 Linux 主机内的用户，在 FreeIPA 界面内点点点添加一个用户，设定初始密码，也可以添加 sudo 权限，默认 shell 选 /bin/bash 即可。

添加完用户后，用户登录 FreeIPA 界面需要修改密码，修改完成后，即可在 Linux 主机内登录了。

> 注：FreeIPA 添加的用户不会在 /etc/passwd 中添加记录。

用户可以创建自己的家目录。默认命令行终端提示符可以通过添加以下环境变量来更换：

```bash
export PS1='[\u@\h \W]\$ '
```



## Kerberos 测试

创建一个用户：

```bash
$ sudo ipa user-add xujiyou --first=xujiyou --last=xujiyou --password
```

使用 kinit 登录用户：

```bash
$ sudo kinit xujiyou
```

第一次登录需要修改密码，我改的密码是 xujiyou@password

显示当前用户：

```bash
$ sudo klist
```

这样创建完用户后，在 freeipa 的 web 界面上也可以看到用户了！

如果忘记了密码，可以在 Web 界面中更改！！！



#### 本地管理员模式

登录：

```bash
$ sudo kinit admin
```

进入：

```bash
$ sudo kadmin.local
```

输入 ? 可以查看命令列表。



#### 创建 keytab 文件

除了使用明文密码之外，Kerberos 还允许使用 keytab 密码文件进行登录，在 kadmin.local 中创建 keytab 密码文件：

```
kadmin.local:  xst -norandkey -k /home/admin/xujiyou.keytab xujiyou@HDP.TESTING.COM
```

查看 keytab 文件：

```bash
$ sudo klist -t -k /home/admin/xujiyou.keytab
```

使用密钥文件进行登录：

```bash
$ sudo kinit -kt /home/admin/xujiyou.keytab xujiyou
```



## 卸载

在 freeipa-1 上执行：

```bash
$ sudo ipa-replica-manage del freeipa-2.hdp.testing.com
```

在 freeipa-2 上执行：

```bash
$ sudo ipa-server-install --uninstall
```

在 freeipa-1 上执行：

```bash
$ sudo ipa-server-install --uninstall
```























