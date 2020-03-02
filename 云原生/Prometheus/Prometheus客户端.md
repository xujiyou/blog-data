# Prometheus 客户端

官方Golang Prometheus Client 业务代码流程:

1. 创建 Prometheus Metric数据项,可export被其他包可以访问
2. 注册定义好的Metric 相当于执行SQL create table
3. 业务在无代码中想插入对时序书库TSDB数据想的数据写入操作,相当与时序SQL insert. 这部分代码可以放在自己的其他包的业务逻辑中去
4. 提供HTTP API接口,让Prometheus 主程序定时来收集TSDB数据.

## Go 客户端示例 - Gin App 统计API请求数量

#### 第一步:初始换监控项Metric

创建 stat/prometheus.go :

```go
package stat

import "github.com/prometheus/client_golang/prometheus"

var (
	GaugeVecApiDuration = prometheus.NewGaugeVec(prometheus.GaugeOpts{
		Name: "apiDuration",
		Help: "api耗时单位ms",
	}, []string{"WSorAPI"})
	GaugeVecApiMethod = prometheus.NewGaugeVec(prometheus.GaugeOpts{
		Name: "apiCount",
		Help: "各种网络请求次数",
	}, []string{"method"})
	GaugeVecApiError = prometheus.NewGaugeVec(prometheus.GaugeOpts{
		Name: "apiErrorCount",
		Help: "请求api错误的次数type: api/ws",
	}, []string{"type"})
)

func init() {
	prometheus.MustRegister(GaugeVecApiDuration, GaugeVecApiMethod, GaugeVecApiError)
}
```

#### 第二步:在业务代码中采集数据

创建 handle/mw_prometheus_http.go ,统计 API 请求持续的时间和 Method 的计数,代码如下：

```go
package handler

import (
	"github.com/gin-gonic/gin"
	"second/stat"
	"time"
)

func MwPrometheusHttp(c *gin.Context) {
	start := time.Now()
	method := c.Request.Method
	stat.GaugeVecApiMethod.WithLabelValues(method).Inc()

	c.Next()

	end := time.Now()
	d := end.Sub(start)
	stat.GaugeVecApiDuration.WithLabelValues(method).Set(float64(d))
}
```

创建 handler/helper.go ：但是没用到这个

```go
package handler

import (
	"github.com/gin-gonic/gin"
	"second/stat"
)

func jsonError(c *gin.Context, msg interface{}) {
	stat.GaugeVecApiError.WithLabelValues("API").Inc()
	var ms string
	switch v := msg.(type) {
	case string:
		ms = v
	case error:
		ms = v.Error()
	default:
		ms = ""
	}
	c.AbortWithStatusJSON(200, gin.H{"ok": false, "msg": ms})
}
```

#### 第三步：编写业务代码

编写 main.go ：

```go
package main

import (
	"github.com/gin-gonic/gin"
	"github.com/prometheus/client_golang/prometheus/promhttp"
	"second/handler"
)

func main() {
	router := gin.Default()
	router.GET("/api/ping", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "hello world",
		})
	})
	router.GET("/metrics", gin.WrapH(promhttp.Handler()))
	router.Group("/api").Use(handler.MwPrometheusHttp)

	_ = router.Run("localhost:8000")
}
```

完成后，启动 main.go 

### 第四步 ：配置 Prometheus

添加配置：

```yaml
- job_name:       'my-first-go-app'

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s

    static_configs:
      - targets: ['localhost:8000']
        labels:
          group: 'dev'
```

重新加载配置后，访问 API ：http://localhost:8000/api/ping

查看 自己应用的 metrics ：http://localhost:8000/metrics

使用 PromQL 查询：

```
promhttp_metric_handler_requests_total{job="my-first-go-app"}
```



过程 就是这么个套路了，至于其中的细节还需要学习。