# Prometheus 概念

Prometheus 是一个时序数据库。

## 数据模型

Prometheus 在底层将所有数据都按时间序列排序，

### 名称和标签

每个时间都是根据 名称（name） 和 可选的键值对（称作标签 label） 来唯一区分的，名称的命名规则是 \[a-zA-Z_:][a-zA-Z0-9_:]* ，跟 Java 变量的命名要求差不多，另外，冒号是保留的。label 命名规范也是这个，不过第一个下划线是保留的。

标签值可以包含所有Unicode字符

标签值为空被认为等同于不存在。

例如`http_requests_total` 表示接收到的HTTP请求总数。

### 样本（samples）

样本表示实际的值，每个样本都包括：

- 一个 float64 值
- 毫秒精度的时间戳

### 符号

给定度量标准名称和一组标签，通常使用以下符号来标识时间序列：

```
<metric name>{<label name>=<label value>, ...}
```

例如，度量时间序列，名称为`api_http_requests_total`，标签是`method="POST"`和`handler="/messages"的可以这样写：

```
api_http_requests_total{method="POST", handler="/messages"}
```

这与[OpenTSDB](http://opentsdb.net/)使用的符号相同。



## 指标类型

Prometheus 有四种指标类型

### 计数器(Counter)

是一个单调递增的数字，只能加，不能减，也可以归0.

可以使用计数器来表示已服务请求，已完成任务或错误的数量。

### Gauge

可任意上升和下降。

通常用于测量值，例如温度或当前的内存使用量，还用于可能上升和下降的“计数”，例如并发请求数。

### 直方图（Histogram）

将计数放到一个一个的桶中。

直方图讲解：https://cloud.tencent.com/developer/article/1495303

### 摘要（Summary）

类似于直方图，就是加入了百分位。



## JOBS AND INSTANCES

作业和实例

对 Prometheus 来说，抓取数据的端点叫做实例，通常对应于单个进程，具有相同目的的实例的集合（比如mysql集群），则成为 jobs。

例如：

job: `api-server`

- instance 1: `1.2.3.4:5670`
- instance 2: `1.2.3.4:5671`
- instance 3: `5.6.7.8:5670`
- instance 4: `5.6.7.8:5671`

### 自动生成标签和时间序列

当 Peometheus 抓取目标时，会自动在抓取的时间序列上添加一些标签，以识别被抓取的目标：

- `job`：目标所属的已配置作业名称。
- `instance`：`:`抓取的目标网址的一部分。









