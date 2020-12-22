# DNS 查询

文档地址：http://www.ruanyifeng.com/blog/2016/06/dns.html

DNS （Domain Name System 的缩写）的作用非常简单，就是根据域名查出IP地址。

## dig 命令

dig 可以显示整个查询过程：

```bash
$ dig www.baidu.com

; <<>> DiG 9.10.6 <<>> www.baidu.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 32614
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;www.baidu.com.			IN	A

;; Query time: 0 msec
;; SERVER: 10.28.100.100#53(10.28.100.100)
;; WHEN: Fri Feb 28 09:47:09 CST 2020
;; MSG SIZE  rcvd: 42
```

指定 DNS 服务器地址及端口，这里的 a 代表 A 记录：

```bash
$ dig @localhost -p 1053 a whoami.example.org

; <<>> DiG 9.10.6 <<>> @localhost -p 1053 a whoami.example.org
; (2 servers found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 40
;; flags: qr aa rd; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 3
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;whoami.example.org.		IN	A

;; ADDITIONAL SECTION:
whoami.example.org.	0	IN	AAAA	::1
_udp.whoami.example.org. 0	IN	SRV	0 0 58102 .

;; Query time: 8 msec
;; SERVER: ::1#1053(::1)
;; WHEN: Fri Feb 28 09:42:31 CST 2020
;; MSG SIZE  rcvd: 135
```

追踪 DNS 查询路径：

```bash
$  dig +trace math.stackexchange.com

; <<>> DiG 9.9.4-RedHat-9.9.4-74.el7_6.2 <<>> +trace math.stackexchange.com
;; global options: +cmd
.                       518400  IN      NS      k.root-servers.net.
.                       518400  IN      NS      i.root-servers.net.
.                       518400  IN      NS      b.root-servers.net.
.                       518400  IN      NS      m.root-servers.net.
.                       518400  IN      NS      c.root-servers.net.
.                       518400  IN      NS      a.root-servers.net.
.                       518400  IN      NS      l.root-servers.net.
.                       518400  IN      NS      d.root-servers.net.
.                       518400  IN      NS      e.root-servers.net.
.                       518400  IN      NS      f.root-servers.net.
.                       518400  IN      NS      j.root-servers.net.
.                       518400  IN      NS      h.root-servers.net.
.                       518400  IN      NS      g.root-servers.net.
;; Received 447 bytes from 10.28.100.100#53(10.28.100.100) in 9 ms

com.                    172800  IN      NS      m.gtld-servers.net.
com.                    172800  IN      NS      e.gtld-servers.net.
com.                    172800  IN      NS      a.gtld-servers.net.
com.                    172800  IN      NS      k.gtld-servers.net.
com.                    172800  IN      NS      b.gtld-servers.net.
com.                    172800  IN      NS      c.gtld-servers.net.
com.                    172800  IN      NS      l.gtld-servers.net.
com.                    172800  IN      NS      d.gtld-servers.net.
com.                    172800  IN      NS      h.gtld-servers.net.
com.                    172800  IN      NS      f.gtld-servers.net.
com.                    172800  IN      NS      i.gtld-servers.net.
com.                    172800  IN      NS      j.gtld-servers.net.
com.                    172800  IN      NS      g.gtld-servers.net.
com.                    86400   IN      DS      30909 8 2 E2D3C916F6DEEAC73294E8268FB5885044A833FC5459588F4A9184CF C41A5766
com.                    86400   IN      RRSIG   DS 8 1 86400 20200311170000 20200227160000 33853 . oSvl9xafooPJcWETNFLKB6qmVMWepYSuo4NX/Yik7hWjP2oTKmWF7aik 8n/iIWA/fl+3NnbjAjZwj78DXbkxAnVRO2Reeh9FVtWapncOVD+yq1oC RGds5pUu+EWh3geMZfVETcSz9P6NgmlN9t/jNQFZHXBOp1JyAjX8MJEm ERE0HOEKjhHsKJELcpHFUHRu8Mbpxuxo3eyULqPJwbluOjToMluLphaB 7qlkIs+XcX1/dltJdRu5ueX4vv51L3IivixCtzZlFTr4SZOshpUc1e2S zB3TAKwN/ne6Tlj4i4i1gVI91IeKqZUpX07ppxzKZ7AuzX26xaGO3hKE E4bkSQ==
;; Received 1185 bytes from 192.36.148.17#53(i.root-servers.net) in 226 ms

stackexchange.com.      172800  IN      NS      ns-925.awsdns-51.net.
stackexchange.com.      172800  IN      NS      ns-1029.awsdns-00.org.
stackexchange.com.      172800  IN      NS      ns-cloud-d1.googledomains.com.
stackexchange.com.      172800  IN      NS      ns-cloud-d2.googledomains.com.
CK0POJMG874LJREF7EFN8430QVIT8BSM.com. 86400 IN NSEC3 1 1 0 - CK0Q1GIN43N1ARRC9OSM6QPQR81H5M9A NS SOA RRSIG DNSKEY NSEC3PARAM
CK0POJMG874LJREF7EFN8430QVIT8BSM.com. 86400 IN RRSIG NSEC3 8 2 86400 20200302054943 20200224043943 56311 com. G8dklsF4RlT6p0d3FsjNfd8sVdOGTdaojV5L2BjYh2nyj0D3+wvU2016 MuhRg1xzoALC/mSmjDFst2JwJlBarXPL59349jP1p3jZ4Ma4/cqxGS3A 8GbOedcGy+iiSqNIP9c11JmG54JmRzsqAuTZDuTXJCriFh64u1uC9eFw KBpgJ7AYdpZQ4R6SO0h7SrRdhpmeAf0uZfDOLsWP6P8eaw==
4OTJBPMM3103AJD1H5IULI2BU3A4BU6A.com. 86400 IN NSEC3 1 1 0 - 4OTJPND3H2KD9P4RE57195J9UCA7J68Q NS DS RRSIG
4OTJBPMM3103AJD1H5IULI2BU3A4BU6A.com. 86400 IN RRSIG NSEC3 8 2 86400 20200303052705 20200225041705 56311 com. I1eaNdFMh3Mv56cH/NrWfUGoWPzHhFZGHAaU2SxpBR+3REbt/ilje59u OkwOeQhKd3HGnqsJYs2vU7acWXNAp3R3wWF4/W5kotYlEjMqbVc9Xuso RTLzVJPp2GzvF661uxdqjj96sm++88Cd8U2QMzxrM5I+e5T0zf2CFi6o TeoN6BoFCD/Jum9W10s6viAa6QhKnsuPq9Zf+ocTWYe9nw==
;; Received 823 bytes from 192.26.92.30#53(c.gtld-servers.net) in 196 ms

math.stackexchange.com. 3600    IN      A       151.101.1.69
math.stackexchange.com. 3600    IN      A       151.101.65.69
math.stackexchange.com. 3600    IN      A       151.101.129.69
math.stackexchange.com. 3600    IN      A       151.101.193.69
;; Received 115 bytes from 216.239.34.109#53(ns-cloud-d2.googledomains.com) in 72 ms
```



## DNS 记录类型

- A：记录地址（Address），返回域名指向的地址
- NS：域名服务器记录（Name Server），返回保存的下一级域名信息的服务器地址，该记录只能设置域名，不能设置 IP 地址。
- MX：邮件记录（Mail eXchange），返回接收电子邮件的服务器地址。
- CNAME：规范名称记录（Canonical Name），返回另一个域名，即当前查询的域名是另一个域名的跳转。
- PTR：逆向查询记录（Pointer Record），只用于从 IP 地址查询域名。

一般来说，为了服务的安全可靠，至少应该有两条`NS`记录，而`A`记录和`MX`记录也可以有多条，这样就提供了服务的冗余性，防止出现单点失败。

由于`CNAME`记录就是一个替换，所以域名一旦设置`CNAME`记录以后，就不能再设置其他记录了（比如`A`记录和`MX`记录）

