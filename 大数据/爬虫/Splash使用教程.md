# Splash 使用教程

Splash 用来执行爬取的 JS 代码，可渲染，也可直接获取到变量值，就跟在 Chrome console 执行 JS 一样！

## 安装方法

首先安装 Sqlash 服务，Sqlash 最好用 Docker 安装，这里我使用 K8s 部署的：

splash-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: app
  name: splash
  namespace: ns-app
spec:
  type: NodePort
  ports:
  - name: "8050"
    port: 8050
    targetPort: 8050
    protocol: TCP
    nodePort: 30001
  selector:
    app: splash
```

splash-deployment.yaml

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app: my-app
  name: splash
  namespace: ns-app
spec:
  replicas: 1
  strategy: {}
  template:
    metadata:
      labels:
        app: splash
    spec:
      containers:
      - image: scrapinghub/splash:3.2
        name: splash
        ports:
        - containerPort: 8050
        resources: {}
      restartPolicy: Always
status: {}
```

然后创建命名空间 `kubectl create namespace ns-app` 再执行 `kubectl apply -f .` 即可，验证安装完成：

```
$ kubectl get pods -n ns-app
NAME                      READY   STATUS    RESTARTS   AGE
splash-675d87bdfb-x87v6   1/1     Running   0          26h
kubectl get svc -n ns-app
NAME     TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
splash   NodePort   10.191.47.80   <none>        8050:30001/TCP   26h
```

这样就可以使用了，访问 Pod 所在宿主机的 ip 加上端口 30001 即可，比如我的是：http://fueltank-1:30001

## 使用方法

我这里是使用的 Scrapy 搭配 Splash 来使用的。

首先安装客户端库 `pip3 install scrapy-splash`

然后在 setting.py 中加入以下代码：

```python
DOWNLOADER_MIDDLEWARES = {
   # 'yarn_spider.middlewares.YarnSpiderDownloaderMiddleware': 543,
    'scrapy_splash.SplashCookiesMiddleware': 723,
    'scrapy_splash.SplashMiddleware': 725,
    'scrapy.downloadermiddlewares.httpcompression.HttpCompressionMiddleware': 810,
}
SPLASH_URL = 'http://fueltank-1:30001'
DUPEFILTER_CLASS = 'scrapy_splash.SplashAwareDupeFilter'
HTTPCACHE_STORAGE = 'scrapy_splash.SplashAwareFSCacheStorage'
```

OK，可以写代码了。爬虫代码：

```python
# -*- coding: utf-8 -*-
import json

import scrapy
from scrapy_splash import SplashRequest


class YarnSpider(scrapy.Spider):
    name = 'yarn'
    allowed_domains = ['fueltank-2.cloud.bbdops.com']
    lua_script = """
        function main(splash)
            assert(splash:go(splash.args.url))
            splash:wait(0.1)
            local result = splash:evaljs("appsTableData")
            return result
        end
        """

    def start_requests(self):
        yield SplashRequest(
            "http://10.28.102.152:8088/cluster/apps/RUNNING", self.parse,
            endpoint='execute',
            args={
                'lua_source': self.lua_script
            }
        )

    def parse(self, response):
        # print(response.body)
        cell_list = json.loads(response.body)
        # print(cell_list)
        for cell in cell_list:
            app_id = cell[0].split(">")[1].split("<")[0]
            user = cell[1]
            name = cell[2]
            application_type = cell[3]
            queue = cell[4]
            start_time = cell[5]
            finish_time = cell[6]
            state = cell[7]
            final_status = cell[8]
            running_containers = cell[9]
            allocated_cpu_vcores = cell[10]
            allocated_memory_mb = cell[11]
            tracking_ui_url = cell[13].split("href='")[1].split("'")[0]
            json_str = json.dumps({"app_id": app_id, "user": user, "name": name, "application_type": application_type,
                                   "queue": queue, "start_time": start_time, "finish_time": finish_time, "state": state,
                                   "final_status": final_status, "running_containers": running_containers,
                                   "allocated_cpu_vcores": allocated_cpu_vcores,
                                   "allocated_memory_mb": allocated_memory_mb,
                                   "tracking_ui_url": tracking_ui_url
                                   }).encode("utf-8").decode('unicode_escape')

            print(json_str)



```

python 里面写了一段 lua 脚本。

具体使用方式看官方教程：https://splash.readthedocs.io/en/stable/

