# Etcd API (一) - 介绍

---

etcd v3使用 [gRPC](https://www.grpc.io/)作为其消息传递协议。etcd项目包括一个基于gRPC的 [Go客户端](https://github.com/coreos/etcd/tree/master/clientv3)和一个命令行实用程序 [etcdctl](https://github.com/coreos/etcd/tree/master/etcdctl)，用于通过gRPC与etcd集群进行通信。对于不支持gRPC的语言，etcd提供了JSON [gRPC网关](https://github.com/grpc-ecosystem/grpc-gateway)。该网关提供一个RESTful代理，该代理将HTTP / JSON请求转换为gRPC消息。

详细查看：https://github.com/etcd-io/etcd/blob/master/Documentation/dev-guide/api_grpc_gateway.md

下面是两个使用 curl 访问 v3 版本 API 的示例：

```bash
$ curl -L http://localhost:2379/v3/kv/range   -X POST -d '{"key": "Zm9v"}'
$ curl -L http://localhost:2379/v3/kv/range   -X POST -d '{"key": "Zm9v", "range_end": "Zm9w"}' | json_pp
```



etcdctl 默认使用 V2 版本的 API ，可以通过设置环境变量来开启 V3 版本的 API：

```bash
$ export ETCDCTL_API=3
```

在本地直接使用 etcd 命令可以启动一个单机 etcd，我们使用这个单机 etcd 来学习其 API 

除了使用 curl 和 etcdctl 访问 etcd v3 API，还可以通过编程语言来访问，具体可看： [etcd-golang客户端.md](../etcd-golang客户端.md) 



---



既然 etcd v3 版本的 API 都是 gRPC 服务，那就从 proto 文件下手，来探索 etcd v3 API。

下载 etcd 源代码。找到 `etcdserver/etcdserverpb/rpc.proto` 文件。

通过 goland 来查看这个 proto 文件的结构，可以看到：

![image-20200319135818836](../../../resource/image-20200319135818836.png)

一共有 6 个 Service。

每个 Service 下面都有若干个 RPC 方法，每个 RPC 方法都有自己的 Request 和 Response 结构。

之后的文章来学习每个方法的 Request 和 Response 结构体，还有 etcdctl ，curl，goland 的客户端的方法调用。















