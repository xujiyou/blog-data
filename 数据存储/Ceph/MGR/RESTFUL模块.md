# MGR 之 RESTFUL 模块

官方文档：https://ceph.readthedocs.io/en/latest/mgr/restful/

RESTful模块通过SSL保护的连接提供REST API对群集状态的访问。



## 开启

通过以下命令开启 RESTFUL 模块：

```bash
$ ceph mgr module enable restful
```

在 RESTFUL 模块可用之前，还需要在下面配置SSL证书。默认情况下，模块将在主机上所有 IPv4 和 IPv6 地址上的端口8003上接受HTTPS请求。



## 安全

所有与Restful的连接均通过SSL保护。可以使用以下命令生成自签名证书：

```bash
$ ceph restful create-self-signed-cert
```

访问：

```bash
$ curl -k https://localhost:8003/
```

生成正规证书：

```bash
$ openssl req -new -nodes -x509 \
  -subj "/O=IT/CN=ceph-mgr-restful" \
  -days 3650 -keyout restful.key -out restful.crt -extensions v3_ca
```

其中，v3_ca 是正规的证书颁发机构，restful.key 是生成的私钥，restful.crt 是证书，导入到 Ceph 中：

``` bash
$ ceph config-key set mgr/restful/$name/crt -i restful.crt
$ ceph config-key set mgr/restful/$name/key -i restful.key
```

其中，`$name`是 ceph-mgr 实例的名称（通常是主机名）。如果所有管理器实例都共享同一证书，则可以省略`$name`部分：

```bash
$ ceph config-key set mgr/restful/crt -i restful.crt
$ ceph config-key set mgr/restful/key -i restful.key
```



## 配置 IP 和端口

默认情况下，RESTFUL 模块使用本机所有的 IP，并且使用 8003 端口。

由于每个ceph-mgr都拥有自己的restful实例，因此可能有必要单独配置它们。可以通过配置密钥工具来更改IP和端口：

```bash
$ ceph config set mgr mgr/restful/$name/server_addr $IP
$ ceph config set mgr mgr/restful/$name/server_port $PORT
```

或者给集群的所有 ceph-mgr 都配置相同的：

```bash
$ ceph config set mgr mgr/restful/server_addr $IP
$ ceph config set mgr mgr/restful/server_port $PORT
```



## 创建一个用户

使用以下命令创建一个 API 用户：

```bash
ceph restful create-key <username>
```

比如：

```bash
$ ceph restful create-key bbders
```

这个命令会生成一个 UUID，也可以使用命令来查看这个 UUID：

```bash
$ ceph restful list-keys
```

使用以下命令来访问 API：

```bash
$ curl -k https://bbders:f7eba59a-76c8-4e79-91ce-f473986d90ec@test-kubenode-1:8003/server
```



## 负载均衡

只有在那时处于活动状态的管理器上才会启动restful。查询Ceph集群状态以查看哪个管理器处于活动状态（例如ceph mgr dump）。为了通过一致的URL使API可用，而不管当前哪个管理器守护程序处于活动状态，您可能希望设置一个负载平衡器前端，以将流量定向到任何可用的管理器端点。



## API

可以通过 `/doc` 来查看所有 API 的用法，比如：

```bash
$ curl -k https://bbders:f7eba59a-76c8-4e79-91ce-f473986d90ec@test-kubenode-1:8003/doc
```

可以使用 API 将 id 为 1 的 OSD 设置为 UP：

```bash
$ echo -En '{"up": true}' | curl --request PATCH --data @- --silent --insecure --user <user> 'https://<ceph-mgr>:<port>/osd/1'
```

一些 API 包括：

- `/config/cluster`: **GET**
- `/config/osd`: **GET**, **PATCH**
- `/crush/rule`: **GET**
- `/mon`: **GET**
- `/osd`: **GET**
- `/pool`: **GET**, **POST**
- `/pool/<arg>`: **DELETE**, **GET**, **PATCH**
- `/request`: **DELETE**, **GET**, **POST**
- `/request/<arg>`: **DELETE**, **GET**
- `/server`: **GET**























