# Ranger 启用 HA

Ranger 管理着 HDP 各个组件的权限，还是比较关键的，启用 HA 保证服务可用。

因为集群开启了 Kerberos，所以这里使用带 SSL 版本的 HA。

官方文档：https://docs.cloudera.com/HDPDocuments/HDP3/HDP-3.1.5/fault-tolerance/content/configuring_ranger_admin_ha.html



## 编译安装 httpd

在 ct3.testing.com 中执行。

依次执行：

```
cd /usr/local
wget https://archive.apache.org/dist/httpd/httpd-2.4.16.tar.gz
wget https://archive.apache.org/dist/apr/apr-1.5.2.tar.gz 
wget https://archive.apache.org/dist/apr/apr-util-1.5.4.tar.gz
tar -xvf httpd-2.4.16.tar.gz
tar -xvf apr-1.5.2.tar.gz 
tar -xvf apr-util-1.5.4.tar.gz
mv apr-1.5.2/ apr
mv apr httpd-2.4.16/srclib/ 
mv apr-util-1.5.4/ apr-util
mv apr-util httpd-2.4.16/srclib/
```

安装编译工具：

```
yum install pcre pcre-devel
yum install gcc
```

编译并安装：

```
cd /usr/local/httpd-2.4.16
./configure
make
make install
```

启动：

```
cd /usr/local/apache2/bin
./apachectl start
curl localhost
```

## 配置

修改配置：

```
cd /usr/local/apache2/conf
cp httpd.conf ~/httpd.conf.backup
vi /usr/local/apache2/conf/httpd.conf
```

在配置文件中去掉以下代码的注释：

```
Listen 443
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so
LoadModule proxy_ajp_module modules/mod_proxy_ajp.so
LoadModule proxy_balancer_module modules/mod_proxy_balancer.so
LoadModule slotmem_shm_module modules/mod_slotmem_shm.so
LoadModule lbmethod_byrequests_module modules/mod_lbmethod_byrequests.so
LoadModule lbmethod_bytraffic_module modules/mod_lbmethod_bytraffic.so
LoadModule lbmethod_bybusyness_module modules/mod_lbmethod_bybusyness.so
LoadModule ssl_module modules/mod_ssl.so
```

然后在文件末尾，加入以下行：

```
Include /usr/local/apache2/conf/ranger-lb-ssl.conf
```

再创建配置文件：

```
vi ranger-lb-ssl.conf
```

内容如下：

```
Listen 443
<VirtualHost *:443>
        ProxyRequests off
        ProxyPreserveHost on

        Header add Set-Cookie "ROUTEID=.%{BALANCER_WORKER_ROUTE}e; path=/" env=BALANCER_ROUTE_CHANGED

        <Proxy balancer://rangercluster>
                BalancerMember http://ct3.testing.com:6080 loadfactor=1 route=1
                BalancerMember http://ct4.testing.com:6080 loadfactor=1 route=2

                Order Deny,Allow
                Deny from none
                Allow from all

                ProxySet lbmethod=byrequests scolonpathdelim=On stickysession=ROUTEID maxattempts=1 failonstatus=500,501,502,503 nofailover=Off
        </Proxy>

        # balancer-manager
        # This tool is built into the mod_proxy_balancer
        # module and will allow you to do some simple
        # modifications to the balanced group via a gui
        # web interface.
        <Location /balancer-manager>
                SetHandler balancer-manager
                Order deny,allow
                Allow from all
        </Location>


       ProxyPass /balancer-manager !
       ProxyPass / balancer://rangercluster/
       ProxyPassReverse / balancer://rangercluster/

</VirtualHost>
```



## 准备证书

```
cd /tmp
openssl genrsa -out server.key 2048
openssl req -new -key server.key -out server.csr
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
openssl pkcs12 -export -passout pass:ranger -in server.crt -inkey server.key -out lbkeystore.p12 -name httpd.lb.server.alias

keytool -importkeystore -deststorepass ranger -destkeypass ranger -destkeystore httpd_lb_keystore.jks -srckeystore lbkeystore.p12 -srcstoretype PKCS12 -srcstorepass ranger -alias httpd.lb.server.alias

keytool -export -keystore httpd_lb_keystore.jks -alias httpd.lb.server.alias -file httpd-lb-trust.cer
```



```
cp server.crt /usr/local/apache2/conf/
cp server.key /usr/local/apache2/conf/
```



## 重启

所有东西准备好之后重启 httpd：

```
cd /usr/local/apache2/bin
./apachectl restart
```



## 在有 Kerberos 的环境中配置 Ranger HA

将上面的 `/tmp/httpd-lb-trust.cer` 拷贝至各个机器。

在每个机器上执行：

```bash
$ keytool -import -file /tmp/httpd-lb-trust.cer -alias httpd.lb.server.alias -keystore /etc/ranger/usersync/conf/mytruststore.jks -storepass changeit
```

添加以下 HTTP Principal Ranger Admin 和负载均衡节点的 `/etc/security/keytabs/spnego.service.keytab`

```
kadmin.local: ktadd -norandkey -kt /etc/security/keytabs/spnego.service.keytab HTTP/ <host3>@EXAMPLE.COM
kadmin.local: ktadd -norandkey -kt /etc/security/keytabs/spnego.service.keytab HTTP/ <host2>@EXAMPLE.COM
kadmin.local: ktadd -norandkey -kt /etc/security/keytabs/spnego.service.keytab HTTP/ <host1>@EXAMPLE.COM
```





## 启用 Ranger Admin HA

在 ranger 界面右上角点击 enable ranger admin ha按钮。

负载均衡地址填 http://ct3.testing.com ，主机是上面配置 httpd 的主机，端口是 80（这里官方网站弄错了）。

完成后，访问：http://ct3.testing.com













