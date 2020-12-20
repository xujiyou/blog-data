# Nginx 安全



## 隐藏 Nginx 版本号

添加配置：

```
server_tokens off;
```



## Nginx 防止 DDos 和 cc 等流量攻击

#### 限制同一时间段ip访问次数

nginx可以通过`ngx_http_limit_conn_module`和`ngx_http_limit_req_module`配置来限制ip在同一时间段的访问次数.

**ngx_http_limit_conn_module**：该模块用于限制每个定义的密钥的连接数，特别是单个IP地址的连接数．使用limit_conn_zone和limit_conn指令．

**ngx_http_limit_req_module**：用于限制每一个定义的密钥的请求的处理速率，特别是从一个单一的IP地址的请求的处理速率。使用“泄漏桶”方法进行限制．指令：limit_req_zone和limit_req．



ngx_http_limit_conn_module：限制单个IP的连接数示例：

```nginx
http { 
  limit_conn_zone $binary_remote_addr zone=addr：10m; 
　　 #定义一个名为addr的limit_req_zone用来存储session，大小是10M内存，
  #以$binary_remote_addr 为key,
  #nginx 1.18以后用limit_conn_zone替换了limit_conn,
  #且只能放在http{}代码段．
  ... 
  server { 
    ... 
    location /download/ { 
      limit_conn addr 1; 　　#连接数限制
      #设置给定键值的共享内存区域和允许的最大连接数。超出此限制时，服务器将返回503（服务临时不可用）错误.
　　　　　　　＃如果区域存储空间不足，服务器将返回503（服务临时不可用）错误
    }

```

可能有几个limit_conn指令,以下配置将限制每个客户端IP与服务器的连接数，同时限制与虚拟服务器的总连接数：

```nginx
http { 
  limit_conn_zone $binary_remote_addr zone=perip：10m; 
  limit_conn_zone $server_name zone=perserver：10m 
  ... 
  server { 
    ... 
    limit_conn perip 10; 　　　　 #单个客户端ip与服务器的连接数．
    limit_conn perserver 100;　　＃限制与服务器的总连接数
    }
```



**ngx_http_limit_req_module：限制某一时间内，单一IP的请求数**．

```nginx
http {
  limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
  ...
　　#定义一个名为one的limit_req_zone用来存储session，大小是10M内存，　　
　　#以$binary_remote_addr 为key,限制平均每秒的请求为1个，
　　#1M能存储16000个状态，rete的值必须为整数，
　　
  server {
    ...
    location /search/ {
      limit_req zone=one burst=5;
　　　　　　　　
　　　　　　　　#限制每ip每秒不超过1个请求，漏桶数burst为5,也就是队列．
　　　　　　　　#nodelay，如果不设置该选项，严格使用平均速率限制请求数，超过的请求被延时处理．
　　　　　　　　#举个栗子：
　　　　　　　　＃设置rate=20r/s每秒请求数为２０个，漏桶数burst为5个，
　　　　　　　　#brust的意思就是，如果第1秒、2,3,4秒请求为19个，第5秒的请求为25个是被允许的，可以理解为20+5
　　　　　　　　#但是如果你第1秒就25个请求，第2秒超过20的请求返回503错误．
　　　　　　　　＃如果区域存储空间不足，服务器将返回503（服务临时不可用）错误　
　　　　　　　　＃速率在每秒请求中指定（r/s）。如果需要每秒少于一个请求的速率，则以每分钟的请求（r/m）指定。　
　　　　　　　　
    }
```

还可以限制来自单个IP地址的请求的处理速率，同时限制虚拟服务器的请求处理速率：

```nginx
http {
  limit_req_zone $binary_remote_addr zone=perip:10m rate=1r/s;
  limit_req_zone $server_name zone=perserver:10m rate=10r/s;
  ...
  server {
    ...
      limit_req zone=perip burst=5 nodelay;　　#漏桶数为５个．也就是队列数．nodelay:不启用延迟．
      limit_req zone=perserver burst=10;　　　　#限制nginx的处理速率为每秒10个
    }
```



#### 禁止ip或ip网段

查找服务器所有访问者ip方法:

```bash
$ awk '{print $1}' nginx_access.log |sort |uniq -c|sort -n
```

nginx.access.log 为nginx访问日志文件所在路径

会到如下结果，前面是ip的访问次数，后面是ip，很明显我们需要把访问次数多的ip并且不是蜘蛛的ip屏蔽掉，如下面结果， 
若 66.249.79.84 不为蜘蛛则需要屏蔽：

```
     89 106.75.133.167
     90 118.123.114.57
     91 101.78.0.210
     92 116.113.124.59
     92 119.90.24.73
     92 124.119.87.204
    119 173.242.117.145
   4320 66.249.79.84
```



#### 屏蔽IP的方法

在nginx的安装目录下面,新建屏蔽ip文件，命名为guolv_ip.conf，以后新增加屏蔽ip只需编辑这个文件即可。 加入如下内容并保存：

```
deny 66.249.79.84 ; 
```

在nginx的配置文件nginx.conf中加入如下配置，可以放到http, server, location, limit_except语句块，需要注意相对路径，本例当中nginx.conf，guolv_ip.conf在同一个目录中。

```
include guolv_ip.conf; 
```

屏蔽ip的配置文件既可以屏蔽单个ip，也可以屏蔽ip段，或者只允许某个ip或者某个ip段访问。

```c
//屏蔽单个ip访问

deny IP; 

//允许单个ip访问

allow IP; 

//屏蔽所有ip访问

deny all; 

//允许所有ip访问

allow all; 

//屏蔽整个段即从123.0.0.1到123.255.255.254访问的命令

deny 123.0.0.0/8

//屏蔽IP段即从123.45.0.1到123.45.255.254访问的命令

deny 124.45.0.0/16

//屏蔽IP段即从123.45.6.1到123.45.6.254访问的命令

deny 123.45.6.0/24

//如果你想实现这样的应用，除了几个IP外，其他全部拒绝，
//那需要你在guolv_ip.conf中这样写

allow 1.1.1.1; 
allow 1.1.1.2;
deny all; 
```

















