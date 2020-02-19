1. 使用Jaeger来对bookinfo应用做分布式跟踪：

   ```
   root@mcdev1:~# kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=jaeger -o jsonpath='{.items[0].metadata.name}') 15032:16686 &
   ```

> Note: 为了演示方便，我们这里使用kubectl命令行工具对jaeger做端口转发，这样就可以通过`http://localhost:15032`来访问jaeger的UI
>
> Note: Istio默认使用样本跟踪率为`1%`，为了确保jaeger收集到分布式跟踪的数据，请使用一下脚本实现多次访问bookinfo应用：

```
root@mcdev1:~# for i in `seq 1 100`; do curl -s -o /dev/null http://$GATEWAY_URL/productpage; done
```

> 然后在jaeger仪表盘的左边区域的服务列表里面选择`productpage.default`，然后点击`Find Traces`，一切正常的话，jaeger的UI会显示最新的20条分布式跟踪的记录数据。

> 随机选择一条分布式跟踪的记录打开，我们会发现每一条jaeger的分布式跟踪链（称为一个trace）是有多个不同的单独请求（称为一个span）组成的树状结构；同时我们也可以打开每个单独的请求来查看请求的状态以及其他的元数据。

1. 使用Grafana来可视化指标度量：

   ```
   root@mcdev1:~# kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=grafana -o jsonpath='{.items[0].metadata.name}') 3000:3000 &
   ```

> Note: 为了演示方便，这里使用kubectl命令行工具对Grafana做端口转发，这样就可以通过`http://localhost:3000/dashboard/db/istio-mesh-dashboard`访问Grafana的UI来查看网格的全局视图以及网格中的服务和工作负载。

> 除了查看网格的全局视图以及网格中的服务和工作负载外，Grafana还可以查看Istio控制层面主要组件Pilot，Galley等运行状况。

1. 使用Kiali来做网格整体可视化：

   ```
   root@mcdev1:~# kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=kiali -o jsonpath='{.items[0].metadata.name}') 20001:20001 &
   ```

> Note: 为了演示方便，我们这里使用kubectl命令行工具对Kiali做端口转发，现在我们可以通过`http://localhost:2001/kiali/`访问Kiali的UI，默认用户名和密码都是`admin`。
>
> 登录Kiali的UI之后会显示Overview页面，这里可以浏览整个服务网格的概况：

![img](https://istio.io/docs/tasks/telemetry/kiali/kiali-overview.png)

> 要查看指定bookinfo的整体服务图，可以点击`default`命名空间卡片，会显示类似于下面的页面，甚至可以动态显示流量走向：