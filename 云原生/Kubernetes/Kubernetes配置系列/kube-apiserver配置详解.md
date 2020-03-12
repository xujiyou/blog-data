# kube-apiserver 配置详解

kube-apiserver 配置官方文档：https://kubernetes.io/zh/docs/reference/command-line-tools-reference/kube-apiserver/

官方文档的配置是没经过分类的。可以通过以下命令来查看分类过后的全部配置选项：

```bash
$ kube-apiserver -h
```

在源码中，kubernetes 的组件的命令行参数都是通过 cobra 来构建的。下面讲解时也会讲配置在源码中的位置。

首先，kube-apiserver 的配置是在 `cmd/kube-apiserver/app/server.go` 中初始化的。

然后，kube-apiserver 的配置分类在 `cmd/kube-apiserver/app/options/options.go` 中。

下面一组一组的学习。我使用的 k8s 版本是 1.17.3

## Generic flags

通用配置的定义全在 `staging/src/k8s.io/apiserver/pkg/server/options/server_run_options.go` 中。

一共有十六个配置，其中有一个特性开关配置。

- **--advertise-address ip** 将apiserver通告给集群成员的IP地址。其余集群必须可以访问此地址。 如果为空，将使用--bind-address。 如果未指定--bind-address，则将使用主机的默认接口。（一般使用默认即可）
- **--cloud-provider-gce-l7lb-src-cidrs cidrs** GCE 的配置，（不用管）
- **--cors-allowed-origins strings **CORS允许的来源清单，以逗号分隔。 允许的来源可以是支持子域匹配的正则表达式。 如果此列表为空，则不会启用CORS。（跨域协议，如果想安全可以设置）
- **--default-not-ready-toleration-seconds int** 等待notReady:NotExecute的秒数,默认300,默认会给所有未设置的toleration的pod设置该值，节点 NotReady 超过这个时间容器就被干掉。（默认即可）
- **--default-unreachable-toleration-seconds int** 同上，unreachable:NoExecute 的容忍秒数，默认情况下添加到尚未具有此容忍度的每个Pod中。 （默认为300）（默认即可）
- **--enable-inflight-quota-handler** 在公平调度和优先调度之间切换 （一般不设置即可）
- **--external-hostname string ** 外部访问 kube-apiserver 时使用的主机名 （设置成本机主机名即可）
- **--livez-grace-period duration** 此选项代表apiserver完成启动并生效所需的最长时间。 从apiserver的启动时间到这段时间为止，这段时间内 /livez 将返回 true （如果能把握好启动时间就设置上）
- **--master-service-namespace string ** DEPRECATED: 过期了，如果不指定 namespace 的话，使用的默认 namespace，默认为 “default” （不要设置）
- **--max-mutating-requests-inflight int** 指定时间内的最大的修改请求数量，如果超过了了这个值，kube-apiserver 会拒绝请求。默认为 200，0表示无限制。（对k8s熟练掌握了，对当前集群了解的话设置）
- **--max-requests-inflight int** 同上，不过请求是非破坏的，默认是 400（对k8s熟练掌握了，对当前集群了解的话设置）
- **--min-request-timeout int** 可选字段，暂时理解为 watch 请求的超时时间，默认值 1800 （可不设置）
- **--request-timeout duration** 可选字段，请求超时时间，这是默认请求超时，对于特定的请求可能会被 --min-request-timeout 覆盖，默认值为1分钟。（可不设置）
- **--shutdown-delay-duration duration** 终止时间设置，在这段时间内，继续处理请求，但不会再接受请求了，相当于黄灯时间，/healthz 返回成功，但是 /readyz 立即返回失败。这可用于允许负载平衡器停止向该服务器发送流量。
- **--target-ram-mb int** 内存限制，以MB为单位，用于配置缓存大小等。

另外，特性开关最后学习。



## Etcd flags



