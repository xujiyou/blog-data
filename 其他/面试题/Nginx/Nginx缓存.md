# Nginx 缓存

网页缓存是由HTTP消息头中的"Cache-control"来控制的，常见的取值有private、no-cache、max-age、must-revalidate等，默认为private。

其作用根据不同的重新浏览方式分为以下几种情况。

| **Cache-directive**                  | **说明**                                                     |
| ------------------------------------ | ------------------------------------------------------------ |
| public                               | 所有内容都将被缓存（客户端和代理服务器都可缓存）。           |
| private                              | 内容只缓存到私有缓存中（仅客户端可以缓存，代理服务器不可缓存）。 |
| no-cache                             | 必须先与服务器确认返回的响应是否被更改，然后才能使用该响应来满足后续对同一个网址的请求。因此，如果存在合适的验证停牌（ETag），no-cache会发起往返通信来验证缓存的响应，如果资源未被更改，可以避免下载。 |
| no-store                             | 所有内容都不会被缓存到缓存或Internet临时文件中，强制缓存和对比缓存都不会触发。 |
| must-revalidation.proxy-revalidation | 如果缓存内容失败，请求必须发送到服务器、代理以进行重新验证。 |
| max-age=xxx(xxx is numeric）         | 缓存的内容将在xxx秒失效，这个选项只在HTTP 1.1可用，并如果和Last-Modified一起使用时，优先级较高。 |



## 缓存规则

默认情况下，NGINX尊重Cache-Control源服务器的标头。它不缓存响应Cache-Control设置为Private，No-Cache或No-Store或Set-Cookie在响应头。NGINX只缓存GET和HEAD客户端请求。

如下配置可覆盖这些默认值：

- proxy_buffering默认为on，若proxy_buffering设置为off，则NGINX不会缓存响应。
- proxy_ignore_headers可以配置忽略Cache-Control：





















































