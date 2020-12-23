# HTTP 之 keep-alive

## Keep-Alive

参考：https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Keep-Alive

含有 `Keep-Alive` 首部的响应示例:

```
Connection: Keep-Alive
Keep-Alive: timeout=5, max=1000
```

- `timeout`：指定了一个空闲连接需要保持打开状态的最小时长（以秒为单位）。需要注意的是，如果没有在传输层设置 keep-alive TCP message 的话，大于 TCP 层面的超时设置会被忽略。超时时间是服务端设置的。
- `max`：在连接关闭之前，在此连接可以发送的请求的最大值。在非管道连接中，除了 0 以外，这个值是被忽略的，因为需要在紧跟着的响应中发送新一次的请求。HTTP 管道连接则可以用它来限制管道的使用。



## Keep-Alive模式

参考：https://www.cnblogs.com/skynet/archive/2010/12/11/1903347.html

我们知道HTTP协议采用“请求-应答”模式，当使用普通模式，即非KeepAlive模式时，每个请求/应答客户和服务器都要新建一个连接，完成之后立即断开连接（HTTP协议为无连接的协议）；当使用Keep-Alive模式（又称持久连接、连接重用）时，Keep-Alive功能使客户端到服务器端的连接持续有效，当出现对服务器的后继请求时，Keep-Alive功能避免了建立或者重新建立连接。

http 1.0中默认是关闭的，需要在http头加入"Connection: Keep-Alive"，才能启用Keep-Alive；http 1.1中默认启用Keep-Alive，如果加入"Connection: close "，才关闭。目前大部分浏览器都是用http1.1协议，也就是说默认都会发起Keep-Alive的连接请求了，所以是否能完成一个完整的Keep-Alive连接就看服务器设置情况。

开启HTTP Keep-Alive之后，能复用已有的TCP链接，当前一个请求已经响应完毕，服务器端没有立即关闭TCP链接，而是等待一段时间接收浏览器端可能发送过来的第二个请求，通常浏览器在第一个请求返回之后会立即发送第二个请求，如果某一时刻只能有一个链接，同一个TCP链接处理的请求越多，开启KeepAlive能节省的TCP建立和关闭的消耗就越多。

当然通常会启用多个链接去从服务器器上请求资源，但是开启了Keep-Alive之后，仍然能加快资源的加载速度。HTTP/1.1之后默认开启Keep-Alive, 在HTTP的头域中增加Connection选项。当设置为`Connection:keep-alive`表示开启，设置为`Connection:close`表示关闭。



## 如何判断消息内容/长度的大小

Keep-Alive模式，客户端如何判断请求所得到的响应数据已经接收完成（或者说如何知道服务器已经发生完了数据）？我们已经知道了，Keep-Alive模式发送玩数据HTTP服务器不会自动断开连接，所有不能再使用返回EOF（-1）来判断（当然你一定要这样使用也没有办法，可以想象那效率是何等的低）！下面我介绍两种来判断方法。



#### 使用消息首部字段Conent-Length

故名思意，Conent-Length表示实体内容长度，客户端（服务器）可以根据这个值来判断数据是否接收完成。但是如果消息中没有Conent-Length，那该如何来判断呢？又在什么情况下会没有Conent-Length呢？



#### 使用消息首部字段Transfer-Encoding

当客户端向服务器请求一个静态页面或者一张图片时，服务器可以很清楚的知道内容大小，然后通过Content-length消息首部字段告诉客户端需要接收多少数据。但是如果是动态页面等时，服务器是不可能预先知道内容大小，这时就可以使用Transfer-Encoding：chunk模式来传输数据了。即如果要一边产生数据，一边发给客户端，服务器就需要使用"Transfer-Encoding: chunked"这样的方式来代替Content-Length。

chunk编码将数据分成一块一块的发生。Chunked编码将使用若干个Chunk串连而成，由一个标明**长度为0**的chunk标示结束。每个Chunk分为头部和正文两部分，头部内容指定正文的字符总数（**十六进制的数字**）和数量单位（一般不写），正文部分就是指定长度的实际内容，两部分之间用**回车换行(CRLF)**隔开。在最后一个长度为0的Chunk中的内容是称为footer的内容，是一些附加的Header信息（通常可以直接忽略）。

参考： [List-Watch原理.md](../../../云原生/Kubernetes/List-Watch原理.md) 



























