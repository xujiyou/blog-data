# Harbor 安装

## 下载 Harbor

去 Github 下载最新版本的 Harbor 离线（offline）版本的安装包。

harbor 默认以来 docker 与 docker-compose



## 证书生成

```bash
$ openssl genrsa -out ca.key 4096
$ openssl req -x509 -new -nodes -sha512 -days 3650 -subj "/C=CN/ST=Chengdu/L=Chengdu/O=BBD/OU=Company/CN=registry.prod.testing.com" -key ca.key -out ca.crt

$ openssl genrsa -out registry.prod.testing.com.key 4096
$ openssl req -sha512 -new -subj "/C=CN/ST=Chengdu/L=Chengdu/O=BBD/OU=Company/CN=registry.prod.testing.com" -key registry.prod.testing.com.key -out registry.prod.testing.com.csr

$ cat > v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1=registry.prod.testing.com
DNS.2=server-04
EOF

$ openssl x509 -req -sha512 -days 3650 -extfile v3.ext -CA ca.crt -CAkey ca.key -CAcreateserial -in registry.prod.testing.com.csr -out registry.prod.testing.com.crt
```

导入到 docker：

```bash
$ sudo mkdir -p /etc/docker/certs.d/registry.prod.testing.com:10443
$ openssl x509 -inform PEM -in registry.prod.testing.com.crt -out registry.prod.testing.com.cert
$ sudo cp registry.prod.testing.com.cert /etc/docker/certs.d/registry.prod.testing.com:10443/
$ sudo cp registry.prod.testing.com.key /etc/docker/certs.d/registry.prod.testing.com:10443/
$ sudo cp ca.crt /etc/docker/certs.d/registry.prod.testing.com:10443/
```

重启 docker：

```bash
$ systemctl restart docker
```



## 配置 harbor

编辑 harbor.yml 文件，内容如下：

```yaml
hostname: registry.prod.testing.com

# http related config
http:
  # port for http, default is 80. If https enabled, this port will redirect to https port
  port: 10080

# https related config
https:
  # https port for harbor, default is 443
  port: 10443
  # The path of cert and key files for nginx
  certificate: /opt/harbor/cert/registry.prod.testing.com.crt
  private_key: /opt/harbor/cert/registry.prod.testing.com.key
  
harbor_admin_password: testing@testing.com

database:
  # The password for the root user of Harbor DB. Change this before any production use.
  password: root123
  # The maximum number of connections in the idle connection pool. If it <=0, no idle connections are retained.
  max_idle_conns: 50
  # The maximum number of open connections to the database. If it <= 0, then there is no limit on the number of open connections.
  # Note: the default number of connections is 1024 for postgres of harbor.
  max_open_conns: 1000

# The default data volume
data_volume: /data3

log:
  # options are debug, info, warning, error, fatal
  level: info
  # configs for logs in local storage
  local:
    # Log files are rotated log_rotate_count times before being removed. If count is 0, old versions are removed rather than rotated.
    rotate_count: 50
    # Log files are rotated only if they grow bigger than log_rotate_size bytes. If size is followed by k, the size is assumed to be in kilobytes.
    # If the M is used, the size is in megabytes, and if G is used, the size is in gigabytes. So size 100, size 100k, size 100M and size 100G
    # are all valid.
    rotate_size: 200M
    # The directory on your host that store log
    location: /data1/harbor-log
```



## 启动 harbor

```bash
$ sudo ./install.sh
```

完成后，浏览器访问：https://registry.prod.testing.com:10443

用户名密码：admin/testing@testing.com

















