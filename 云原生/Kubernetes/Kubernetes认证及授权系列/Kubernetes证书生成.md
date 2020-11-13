# Kubernetes 证书生成

官方文档：https://kubernetes.io/zh/docs/concepts/cluster-administration/certificates/

注意: 证书中的 CN 即为用户名，O 即为组名。

当使用客户端证书进行认证时，可以通过 `easyrsa`、`openssl` 或 `cfssl` 手动生成证书。

下面先学习 openssl 生成证书



## openssl 生成证书

首先，不论是证书还是私钥都可以以 `.pem` 作为文件后缀名。

私钥文件通常以 `.key` 结尾，证书文件通常以 `.crt` 结尾，证书签名请求文件通常以 `.csr` 结尾。

不论什么后缀的文件，其实都是文本格式的文件。

证书文件的开头和结尾是这样子的：

```
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----
```

私钥文件的开头和结尾是这样的：

```
-----BEGIN RSA PRIVATE KEY-----
...
-----END RSA PRIVATE KEY-----
```

证书签名请求文件的开头和结尾是这样的：

```
-----BEGIN CERTIFICATE REQUEST-----
...
-----END CERTIFICATE REQUEST-----
```



#### 生成 CA 证书

在生成证书之前，都需要先创建 CA 证书。

1. 生成位数为 2048 的证书密钥 ca.key：

   ```bash
   $ openssl genrsa -out ca.key 2048
   ```

2. 依据 ca.key 生成 CA 证书 ca.crt （使用 -days 参数来设置证书有效时间）：

   ```bash
   $ openssl req -x509 -new -nodes -key ca.key -subj "/CN=ca_name" -days 10000 -out ca.crt
   ```

这样，CA 证书就创建好了。生成的私钥及证书名是自定义的，使用 `-out` 参数指定，也可以都使用 `.pem` 为后缀。



#### 创建服务证书

有了 CA 证书之后，再来创建服务证书。

1. 先生成密钥位数为 2048 的 server.key：

   ```bash
   $ openssl genrsa -out server.key 2048
   ```

   

2. 在创建证书签名请求文件之前，需要先编写配置文件 server.conf：

   ```toml
   [ req ]
   default_bits = 2048
   prompt = no
   default_md = sha256
   req_extensions = req_ext
   distinguished_name = dn
   
   [ dn ]
   O = group.name
   CN = server.name
   
   [ req_ext ]
   subjectAltName = @alt_names
   
   [ alt_names ]
   DNS.1 = kubernetes
   DNS.2 = kubernetes.default
   DNS.3 = kubernetes.default.svc
   DNS.4 = kubernetes.default.svc.cluster
   DNS.5 = kubernetes.default.svc.cluster.local
   DNS.6 = fueltank-1
   DNS.7 = fueltank-2
   DNS.8 = fueltank-3
   DNS.9 = localhost
   IP.1 = 127.0.0.1
   IP.2 = 172.20.20.162
   IP.3 = 172.20.20.179
   IP.4 = 172.20.20.145
   
   [ v3_ext ]
   authorityKeyIdentifier=keyid,issuer:always
   basicConstraints=CA:FALSE
   #keyUsage=keyEncipherment,dataEncipherment
   keyUsage=digitalSignature,keyEncipherment,dataEncipherment
   extendedKeyUsage=serverAuth,clientAuth
   subjectAltName=@alt_names
   ```

   其中，`req` 是证书的一些参数，`dn` 是自定义的一些内容， DNS 和 IP 是服务绑定的 IP 和 DNS。

   

3. 再根据私钥和配置文件创建证书签名请求文件 server.csr

   ```bash
   $ openssl req -new -key server.key -config server.conf -out server.csr
   ```

4. 最后使用 ca.key、ca.crt 和 server.csr 、server.conf 来生成服务器证书

   ```bash
   $ openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -days 10000 -extensions v3_ext -extfile server.conf -out server.crt
   ```
   


完成后，会生成一个名为 `server.crt` 的证书文件，同时还创建了一个 `ca.srl` 文件。

这个 ca.srl 文件是CA使用的序列号文件，当使用"-CA"选项来签名时，它将会使用某个文件中指定的序列号来唯一标识此次签名后的证书文件。这个序列号文件的内容仅只有一行，这一行的值为16进制的数字，当某个序列号被使用后，该文件中的序列号将自动增加。

默认序列号文件以CA证书文件基名加".srl"为后缀命名。如CA证书为"mycert.pem"，则默认寻找的序列号文件为"mycert.srl"

测试一下：

```bash
$ cat ca.srl 
8C135180FF2AB966
$ #重新执行一下上边生成证书的命令
$ openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -days 10000 -extensions v3_ext -extfile server.conf -out server.crt
$ cat ca.srl 
8C135180FF2AB967
```

果然增长了 1。



## 使用证书生成 secret

```bash
$ kubectl -n 	s1-search create secret tls tls-secret --cert=server.crt --key=server.key
```

在 ingress 里面这样使用：

```
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: gateway
  namespace: s1-search
spec:
  rules:
    - host: gateway.s1-search.tech
      http:
        paths:
          - path: /
            backend:
              serviceName: cloud-gateway-server
              servicePort: 8003
  tls:
    - hosts:
        - gateway.s1-search.tech
      secretName: tls-secret
```

