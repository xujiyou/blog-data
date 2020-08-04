# ES 踩坑

ES 总是会出现掉节点的情况，这一秒 up 了，下一秒就要 down，如此循环往复。报错日志也没什么特别的错误，就是那台爱掉的节点一直循环报 no master 的错误。



## 解决方案

默认配置为：节点每隔 1s 同 master 发送1次心跳，超时时间为30s，测试次数为3次，超过3次，则认为该节点同master已经脱离了。以上为elasticsearch的默认配置。在实际生产环境中，每隔1s，太频繁了，会产生太多网络流量。我们可以在elasticsearch.yml如下修改：

```properties
discovery.zen.fd.ping_timeout: 120s  
discovery.zen.fd.ping_retries: 6  
discovery.zen.fd.ping_interval: 30s  
```

超时时间设为2分钟，超过6次心跳没有回应，则认为该节点脱离master，每隔30s发送一次心跳。



另外参考：https://www.elastic.co/guide/en/elasticsearch/reference/7.x/delayed-allocation.html

避免在掉节点后立马就重新平衡数据，可以设置延迟，比如设置5分钟延迟：

```http
PUT _all/_settings
{
  "settings": {
    "index.unassigned.node_left.delayed_timeout": "5m"
  }
}
```

如果在 5 分钟内，节点恢复了，就不用平衡数据了！



## 原因

检测网络延迟和丢包：

````bash
mtr -r 10.28.92.11
````

发现延迟也不大，也没丢包。

后来发现，上述情况发生在 ES 平衡数据时，每次数据快平衡到 100% 后，那个节点就要 down 了。

在 ES 平衡数据时，会有大量的网络传输，在大量网络传输的情况下，ES 节点之间的心掉检测就可能有丢包和延迟了！















