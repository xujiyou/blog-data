# FreeIPA 安装

参考：https://www.sysit.cn/blog/post/sysit/FreeIPA-HA%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE

`FreeIPA`建立在著名的开源组件和标准协议之上，是一个集成的安全信息管理解决方案，具有易于管理、安装和配置任务自动化的特点。它整合了`389-ds（LDAP）`、`Kerberos`、`NTP`、`bind`、`apache`、`tomcat`核心软件包，形成一个以`389-ds（LDAP）`为数据存储后端，`Kerberos`为验证前端，`bind`为主机识别，并且具有统一的命令行管理工具及`apache+tomcat`提供的`web`管理界面的集成信息管理系统。



## 安装

在 CentOS8 上安装 FreeIPA

```bash
$ sudo yum -y install @idm:DL1
$ sudo yum -y install freeipa-server
```

配置：

```bash
$ sudo ipa-server-install
```

过程如下：

```
The log file for this installation can be found in /var/log/ipaserver-install.log
==============================================================================
This program will set up the IPA Server.
Version 4.8.4

This includes:
  * Configure a stand-alone CA (dogtag) for certificate management
  * Configure the NTP client (chronyd)
  * Create and configure an instance of Directory Server
  * Create and configure a Kerberos Key Distribution Center (KDC)
  * Configure Apache (httpd)
  * Configure the KDC to enable PKINIT

To accept the default shown in brackets, press the Enter key.

Do you want to configure integrated DNS (BIND)? [no]: 

Enter the fully qualified domain name of the computer
on which you're setting up server software. Using the form
<hostname>.<domainname>
Example: master.example.com.


Server host name [node1.testing.com]: 

The domain name has been determined based on the host name.

Please confirm the domain name [testing.com]: 

The kerberos protocol requires a Realm name to be defined.
This is typically the domain name converted to uppercase.

Please provide a realm name [TESTING.COM]: 
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

Do you want to configure chrony with NTP server or pool address? [no]: yes
Enter NTP source server addresses separated by comma, or press Enter to skip: 
Enter a NTP source pool address, or press Enter to skip: 

The IPA Master Server will be configured with:
Hostname:       node1.testing.com
IP address(es): 10.10.10.16
Domain name:    testing.com
Realm name:     TESTING.COM

The CA will be configured with:
Subject DN:   CN=Certificate Authority,O=PROD.BBDOPS.COM
Subject base: O=TESTING.COM
Chaining:     self-signed

Continue to configure the system with these values? [no]: yes

The following operations may take some minutes to complete.
Please wait until the prompt is returned.

Disabled p11-kit-proxy
Synchronizing time
No SRV records of NTP servers found and no NTP server or pool address was provided.
Using default chrony configuration.
Attempting to sync time with chronyc.
Time synchronization was successful.
Configuring directory server (dirsrv). Estimated time: 30 seconds
  [1/44]: creating directory server instance
  [2/44]: configure autobind for root
  [3/44]: stopping directory server
  [4/44]: updating configuration in dse.ldif
  [5/44]: starting directory server
  [6/44]: adding default schema
  [7/44]: enabling memberof plugin
  [8/44]: enabling winsync plugin
  [9/44]: configure password logging
  [10/44]: configuring replication version plugin
  [11/44]: enabling IPA enrollment plugin
  [12/44]: configuring uniqueness plugin
  [13/44]: configuring uuid plugin
  [14/44]: configuring modrdn plugin
  [15/44]: configuring DNS plugin
  [16/44]: enabling entryUSN plugin
  [17/44]: configuring lockout plugin
  [18/44]: configuring topology plugin
  [19/44]: creating indices
  [20/44]: enabling referential integrity plugin
  [21/44]: configuring certmap.conf
  [22/44]: configure new location for managed entries
  [23/44]: configure dirsrv ccache and keytab
  [24/44]: enabling SASL mapping fallback
  [25/44]: restarting directory server
  [26/44]: adding sasl mappings to the directory
  [27/44]: adding default layout
  [28/44]: adding delegation layout
  [29/44]: creating container for managed entries
  [30/44]: configuring user private groups
  [31/44]: configuring netgroups from hostgroups
  [32/44]: creating default Sudo bind user
  [33/44]: creating default Auto Member layout
  [34/44]: adding range check plugin
  [35/44]: creating default HBAC rule allow_all
  [36/44]: adding entries for topology management
  [37/44]: initializing group membership
  [38/44]: adding master entry
  [39/44]: initializing domain level
  [40/44]: configuring Posix uid/gid generation
  [41/44]: adding replication acis
  [42/44]: activating sidgen plugin
  [43/44]: activating extdom plugin
  [44/44]: configuring directory to start on boot
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
  [2/29]: Add ipa-pki-wait-running
  [3/29]: reindex attributes
  [4/29]: exporting Dogtag certificate store pin
  [5/29]: stopping certificate server instance to update CS.cfg
  [6/29]: backing up CS.cfg
  [7/29]: disabling nonces
  [8/29]: set up CRL publishing
  [9/29]: enable PKIX certificate path discovery and validation
  [10/29]: starting certificate server instance
  [11/29]: configure certmonger for renewals
  [12/29]: requesting RA certificate from CA
  [13/29]: setting audit signing renewal to 2 years
  [14/29]: restarting certificate server
  [15/29]: publishing the CA certificate
  [16/29]: adding RA agent as a trusted user
  [17/29]: authorizing RA to modify profiles
  [18/29]: authorizing RA to manage lightweight CAs
  [19/29]: Ensure lightweight CAs container exists
  [20/29]: configure certificate renewals
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
  [1/21]: stopping httpd
  [2/21]: backing up ssl.conf
  [3/21]: disabling nss.conf
  [4/21]: configuring mod_ssl certificate paths
  [5/21]: setting mod_ssl protocol list
  [6/21]: configuring mod_ssl log directory
  [7/21]: disabling mod_ssl OCSP
  [8/21]: adding URL rewriting rules
  [9/21]: configuring httpd
Nothing to do for configure_httpd_wsgi_conf
  [10/21]: setting up httpd keytab
  [11/21]: configuring Gssproxy
  [12/21]: setting up ssl
  [13/21]: configure certmonger for renewals
  [14/21]: publish CA cert
  [15/21]: clean up any existing httpd ccaches
  [16/21]: configuring SELinux for httpd
  [17/21]: create KDC proxy config
  [18/21]: enable KDC proxy
  [19/21]: starting httpd
  [20/21]: configuring httpd to start on boot
  [21/21]: enabling oddjobd
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
Configuring client side components
This program will set up IPA client.
Version 4.8.4

Using existing certificate '/etc/ipa/ca.crt'.
Client hostname: kubenode1.prod.bbdops.com
Realm: PROD.BBDOPS.COM
DNS Domain: prod.bbdops.com
IPA Server: kubenode1.prod.bbdops.com
BaseDN: dc=prod,dc=bbdops,dc=com

Configured sudoers in /etc/authselect/user-nsswitch.conf
Configured /etc/sssd/sssd.conf
Systemwide CA database updated.
Adding SSH public key from /etc/ssh/ssh_host_ed25519_key.pub
Adding SSH public key from /etc/ssh/ssh_host_ecdsa_key.pub
Adding SSH public key from /etc/ssh/ssh_host_rsa_key.pub
Could not update DNS SSHFP records.
SSSD enabled
Configured /etc/openldap/ldap.conf
Configured /etc/ssh/ssh_config
Configured /etc/ssh/sshd_config
Configuring prod.bbdops.com as NIS domain.
Client configuration complete.
The ipa-client-install command was successful

Please add records in this file to your DNS system: /tmp/ipa.system.records.i8prjg8l.db
==============================================================================
Setup complete

Next steps:
        1. You must make sure these network ports are open:
                TCP Ports:
                  * 80, 443: HTTP/HTTPS
                  * 389, 636: LDAP/LDAPS
                  * 88, 464: kerberos
                UDP Ports:
                  * 88, 464: kerberos
                  * 123: ntp

        2. You can now obtain a kerberos ticket using the command: 'kinit admin'
           This ticket will allow you to use the IPA tools (e.g., ipa user-add)
           and the web user interface.

Be sure to back up the CA certificates stored in /root/cacert.p12
These files are required to create replicas. The password for these
files is the Directory Manager password
The ipa-server-install command was successful
```

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



## 用户管理

FreeIPA 可以管理 Linux 主机内的用户，在 FreeIPA 界面内点点点添加一个用户，设定初始密码，也可以添加 sudo 权限，默认 shell 选 /bin/bash 即可。

添加完用户后，用户登录 FreeIPA 界面需要修改密码，修改完成后，即可在 Linux 主机内登录了。

> 注：FreeIPA 添加的用户不会在 /etc/passwd 中添加记录。

用户可以创建自己的家目录。默认命令行终端提示符可以通过添加以下环境变量来更换：

```bash
export PS1='[\u@\h \W]\$ '
```

























