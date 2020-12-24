# Istio 实现灰度发布

代码基础是 ： [Istio之Hello-world教程.md](../Istio之Hello-world教程.md) ，下面将在这个基础上进行学习灰度发布。

灰度发布就是将一部分流量导入到其他版本。

---

## 基于流量比例的路由

将百分之二十的流量导入到 v2.0 版本。

下面首先来添加一个 other 服务的 v2.0 版本，很简单，就是改下输出字符串：

```java
package work.xujiyou.other.api;

import io.opentracing.Span;
import io.opentracing.Tracer;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import work.xujiyou.other.config.JaegerTracerHelper;

/**
 * OtherApi class
 *
 * @author jiyouxu
 * @date 2020/2/3
 */
@RestController
@RequestMapping("/api")
public class OtherApi {

    @GetMapping("/other")
    public String other() {
        Tracer tracer = JaegerTracerHelper.initTracer("ApiOtherService");
        Span span = tracer.buildSpan("say-hello").start();
        span.setTag("name", "other");

        String otherStr = String.format("other project v2.0");

        System.out.println(otherStr);
        span.log("Love service say hello");
        span.finish();

        return otherStr;
    }

}
```

就是加了一个 v2.0 而已。加好之后 clean 再编译。

然后重新打包上传：

```bash
#!/bin/bash
#保证 docker login 已经执行过
# sudo docker login --username=552003271@qq.com registry.cn-chengdu.aliyuncs.com
docker -H tcp://fueltank-1:2376 build -t registry.cn-chengdu.aliyuncs.com/bbd-image/other:v2.0  ./
docker -H tcp://fueltank-1:2376 push registry.cn-chengdu.aliyuncs.com/bbd-image/other:v2.0
```

然后创建新版本的 Deployment：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: other-service
  namespace: xujiyou-test
  labels:
    app: other
spec:
  ports:
    - port: 8081
  selector:
    app: other
  type: NodePort

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: other
  namespace: xujiyou-test
  labels:
    app: other
    version: v1.0
spec:
  replicas: 3
  selector:
    matchLabels:
      app: other
      version: v1.0
  template:
    metadata:
      labels:
        app: other
        version: v1.0
    spec:
      containers:
        - name: other
          image: registry.cn-chengdu.aliyuncs.com/bbd-image/other:0.0.1
          ports:
            - containerPort: 8081
---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: other-v2
  namespace: xujiyou-test
  labels:
    app: other
    version: v2.0
spec:
  selector:
    matchLabels:
      app: other
      version: v2.0
  template:
    metadata:
      labels:
        app: other
        version: v2.0
    spec:
      containers:
        - name: other
          image: registry.cn-chengdu.aliyuncs.com/bbd-image/other:v2.0
```

可以删除再重建：

```bash
$ kubectl apply -f other-service-deployment.yaml
```



然后创建 other-service 的 DestinationRule，other-destination-rule.yaml:

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: other
  namespace: xujiyou-test
spec:
  host: other-service
  subsets:
    - name: v1
      labels:
        version: v1.0
    - name: v2
      labels:
        version: v2.0
```

创建：

```bash
$ kubectl apply -f other-destination-rule.yaml
```



然后创建 VirtualService ，other-service-deployment.yaml：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  namespace: xujiyou-test
  name: other
spec:
  hosts:
    - other-service
  http:
    - route:
      - weight: 80
        destination:
          host: other-service
          subset: v1
          port:
            number: 8081
      - weight: 20
        destination:
          host: other-service
          subset: v2
          port:
            number: 8081
```

注意这里的 weight ，就代表了每个规则的权重，这里是四比一。

创建：

```bash
$ kubectl apply -f other-virtual-service.yaml
```



最后待 Pod 创建完成后，访问：

```bash
$ curl http://fueltank-1/api/hello?version=v1
```

多访问几次，会发现差不多就是访问五次出现一次 v2.0 版本的输出！



## 基于请求内容的路由

这一部分已经在  [Istio之Hello-world教程.md](../Istio之Hello-world教程.md)  中实现了。当然也可以将基于比例的路由和基于请求内容的路由进行组合。

## TCP 服务灰度发布 

上边都是 HTTP 的灰度发布，TCP 灰度发布一样的道理，只需要把 http 改成 tcp 就可以了。。

