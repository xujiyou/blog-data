# ES 开启安全认证

---

首先在 elasticsearch.yml 中设置：

```
xpack.security.enabled: true
discovery.type: single-node # 单点需设置，集群不用
```

重启 ES，然后执行：

```bash
$ sudo bin/elasticsearch-keystore create # 如果已经存在文件了，不要重新生成
$ sudo bin/elasticsearch-setup-passwords interactive # 这个操作在集群中只能做一次
```

interactive 是互动的意思，稍后输入各项用户的密码。

然后登录 kibana ，发现需要用户名，输入kibana的用户名及密码，爆 403，使用 elastic 的密码，顺利登录。

---

给 ES 加入证书，YUM 安装方式

执行命令生成CA证书：

```bash
$ sudo bin/elasticsearch-certutil ca
```

期间使用默认文件名，输入密码：123456，之后会在当前目录生成一个 `elastic-stack-ca.p12` 的文件

然后通过CA证书为集群中的每个节点生成证书和私钥：

```bash
$ bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12
```

将产生新文件 `elastic-certificates.p12`

默认情况下 `elasticsearch-certutil` 生成没有主机名信息的证书，这意味着你可以将证书用于集群中的每个节点，另外要关闭主机名验证。

将 elastic-certificates.p12 放到 `/etc/elasticsearch/certs` 中，certs 目录是自己建立的，一定注意这个文件的权限，要保证 elasticsearch 用户能访问到，要不 ES 启动会失败

```bash
$ sudo ls -sl /etc/elasticsearch/certs
4 -rw-rw---- 1 root elasticsearch 2527 1月   9 11:35 elastic-certificates.p12
```

然后修改 `elasticsearch.yml` 文件，加入以下几项：

```
xpack.security.http.ssl.enabled: true
xpack.security.http.ssl.verification_mode: certificate
xpack.security.http.ssl.keystore.path: certs/elastic-certificates.p12
xpack.security.http.ssl.truststore.path: certs/elastic-certificates.p12

xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: certs/elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: certs/elastic-certificates.p12
```

然后使用以下命令为 ES 添加 p12 文件的密码：

```bash
$ sudo bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
$ sudo bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password

$ sudo bin/elasticsearch-keystore add xpack.security.http.ssl.keystore.secure_password
$ sudo bin/elasticsearch-keystore add xpack.security.http.ssl.truststore.secure_password
```

重启ES，这次重启，会花费一段时间。

然后验证：

```bash
$ curl -k https://127.0.0.1:9200/_cat/nodes?v -u elastic:123456
```

完美！

## Kibana 加入 SSL

修改 kibana.yml 

首先 es 的地址要写成 https:

```
elasticsearch.hosts: ["https://fueltank-4:9200"]
```

然后配置 ES 的用户名及密码：

```
elasticsearch.username: "elastic"
elasticsearch.password: "123456"
```

也可以使用密文，执行命令：

```bash
$ sudo bin/kibana-keystore create
$ sudo bin/kibana-keystore add elasticsearch.username
$ sudo bin/kibana-keystore add elasticsearch.password
```

然后修改 kibana.yml:

```
elasticsearch.username: "${elasticsearch.username}"
elasticsearch.password: "${elasticsearch.password}"
```

---

Kibana 本身配置自签名证书。

执行命令生成密钥对：

```bash
$ sudo openssl genrsa -out server.key 1024
$ openssl req -new -x509 -days 3650 -key server.key -out server.crt -subj "/C=CN/ST=mykey/L=mykey/O=mykey/OU=mykey/CN=domain1/CN=domain2/CN=domain3"
```

然后在 kibana.yml 中加入：

```
server.ssl.enabled: true
server.ssl.certificate: /etc/kibana/server.crt
server.ssl.key: /etc/kibana/server.key
```



## Logstash 加入 ES 认证

每个 ES 输出插件的配置要这样写：

```
elasticsearch {
    hosts => ["https://127.0.0.1:9200"]
    index => "log-message-%{+YYYY.MM.dd}"
    user => "${elasticsearch.username}"
    password => "${elasticsearch.password}"
    ssl => true
    ssl_certificate_verification => false
    keystore => '/etc/elasticsearch/certs/elastic-certificates.p12'
    keystore_password => '123456'
    truststore => '/etc/elasticsearch/certs/elastic-certificates.p12'
    truststore_password => '123456'
}
```

