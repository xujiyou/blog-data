# Prometheus 查询

Prometheus 提供了一个实用性的查询语言，名为 PromQL，允许用户实时选择和聚合时间序列数据，表达式的结果既可以显示为图形，也可以显示为表格。

外部系统也可以通过 HTTP Api 调用 PromQL。

---

## 数据类型

一个表达式或子表达式可以计算为以下四种类型之一：

实时向量（**Instant vector** ）：一组时间序列，每个时间序列包含一个样本，所有样本共享相同的时间戳

范围向量（**Range vector**）：一组时间序列，其中包含每个时间序列随时间的一系列数据点

标量（**Scalar**）：一个简单的浮点值

**String**：一个简单的字符串值；目前未使用

## 字面值

### 字符串

字符串可以用单引号、双引号或反勾号指定为文本。



---



## 时间序列选择器

### 即时向量选择器

即时矢量选择器允许在给定的时间戳（即时）下选择一组时间序列和每个样本的单个样本值：以最简单的形式，仅指定度量名称。这将导致一个即时向量，其中包含具有该度量名称的所有时间序列的元素。

本示例选择所有具有`http_requests_total`度量标准名称的时间序列：

```
http_requests_total
```

通过在花括号（`{}`）后面加上逗号分隔的标签匹配器列表，可以进一步过滤这些时间序列。

本示例仅选择那些具有`http_requests_total` 度量标准名称的时间序列，同时将其`job`标签设置为，`prometheus`并将其 `group`标签设置为`canary`：

```
http_requests_total{job="prometheus",group="canary"}
```

也可以否定地匹配标签值，或将标签值与正则表达式匹配。存在以下标签匹配运算符：

- `=`：选择与提供的字符串完全相同的标签。
- `!=`：选择不等于提供的字符串的标签。
- `=~`：选择与提供的字符串进行正则表达式匹配的标签。
- `!~`：选择不与提供的字符串进行正则表达式匹配的标签。

例如，此选择所有`http_requests_total`的时间序列`staging`， `testing`以及`development`环境和HTTP比其他方法`GET`。

```
http_requests_total{environment=~"staging|testing|development",method!="GET"}
```

向量选择器必须指定一个名称或至少一个与空字符串不匹配的标签匹配器。以下表达式是非法的：

```
{job=~".*"} # Bad!
```

相反，这些表达式都是有效的，因为它们都具有与空标签值不匹配的选择器。

```
{job=~".+"}              # Good!
{job=~".*",method="get"} # Good!
```

通过与内部`__name__`标签匹配，标签匹配器也可以应用于度量标准名称 。例如，该表达式`http_requests_total`等效于 `{__name__="http_requests_total"}`。比其他的匹配器`=`（`!=`，`=~`，`!~`）也可以使用。以下表达式选择名称以开头的所有度量`job:`：

```
{__name__=~"job:.*"}
```



---



### 范围向量选择器

范围矢量文字的工作方式与即时矢量文字一样，不同的是它们从当前即时中选择了一系列样本。从句法上讲，范围持续时间附加在向量选择器末尾的方括号中，以指定应为每个结果范围向量元素提取多远的时间值。

持续时间指定为数字，紧随其后的是以下单位之一：

- `s` -秒
- `m` - 分钟
- `h` - 小时
- `d` - 天
- `w` -周
- `y` -年

在此示例中，我们选择所有时间序列在过去5分钟内记录的所有值，这些时间序列的指标名称`http_requests_total`和`job`标签设置为`prometheus`：

```
http_requests_total{job="prometheus"}[5m]
```

并且选择了时间段之后，会显示每一次的增加记录，如：

![image-20200301153316475](../../resource/image-20200301153316475.png)

@ 后面的是时间戳，第一条的时间戳正好是五分钟前。

### 偏移量

所述`offset`改性剂可以改变时间为查询中的个别时刻和范围矢量偏移。

例如，以下表达式返回`http_requests_total`相对于当前查询评估时间的过去5分钟的值 ：

```
http_requests_total offset 5m
```

请注意，`offset`修饰符始终需要立即跟随选择器，即以下内容将是正确的：

```
sum(http_requests_total{method="GET"} offset 5m) // GOOD.
```

虽然以下是*不正确的*：

```
sum(http_requests_total{method="GET"}) offset 5m // INVALID.
```

范围向量的工作原理相同。这将返回`http_requests_total`一周前的5分钟费率 ：

```
rate(http_requests_total[5m] offset 1w)
```

### 注释

PromQL支持以开头的行注释`#`。例：

```
    # This is a comment
```



---



## 操作符

### 算术二进制运算符

Prometheus中存在以下二进制算术运算符：

- `+` （加成）
- `-` （减法）
- `*` （乘法）
- `/` （除）
- `%` （取模）
- `^` （幂）

在标量/标量，向量/标量和向量/向量值对之间定义了二进制算术运算符。

**在两个标量之间**，其行为显而易见：它们求值另一个标量，这是将运算符应用于两个标量操作数的结果。

**在即时向量和标量之间**，将运算符应用于向量中每个数据样本的值。例如，如果时间序列瞬时向量乘以2，则结果是另一个向量，其中原始向量的每个样本值都乘以2。

**在两个即时向量之间**，将二进制算术运算符应用于左侧向量中的每个条目，并将其 应用于右侧向量中的[匹配元素](https://prometheus.io/docs/prometheus/latest/querying/operators/#vector-matching)。结果被传播到结果向量中，并且分组标签成为输出标签集。指标名称已删除。在右侧向量中找不到匹配条目的条目不属于结果。

### 比较二进制运算符

Prometheus中存在以下二进制比较运算符：

- `==` （等于）
- `!=` （不相等）
- `>` （大于）
- `<` （少于）
- `>=` （大于等于）
- `<=` （不等于）

### 逻辑/集合二元运算符

这些逻辑/集合二进制运算符仅在即时向量之间定义：

- `and` （路口）
- `or` （联盟）
- `unless` （补充）

`vector1 and vector2`会产生一个向量，该向量`vector1`由其元素组成， 其中的元素`vector2`具有完全匹配的标签集。其他元素被删除。度量标准名称和值从左侧矢量继承。

`vector1 or vector2`会产生一个向量，其中包含的所有原始元素（标签集+值），`vector1`并且`vector2` 其中所有元素在中都没有匹配的标签集`vector1`。

`vector1 unless vector2`会产生一个向量，该向量由 `vector1`没有`vector2`完全匹配的标签集的元素组成。两个向量中的所有匹配元素都将被删除。

## 集合运算符

Prometheus支持以下内置的聚合运算符，这些运算符可用于聚合单个即时向量的元素，从而产生具有聚合值的较少元素的新向量：

- `sum` （计算尺寸总和）
- `min` （选择最小尺寸）
- `max` （选择最大尺寸）
- `avg` （计算尺寸的平均值）
- `stddev` （计算总体尺寸的标准偏差）
- `stdvar` （计算总体标准方差）
- `count` （计算向量中元素的数量）
- `count_values` （计数具有相同值的元素数）
- `bottomk` （按样本值最小的k个元素）
- `topk` （按样本值最大k个元素）
- `quantile` （计算整个尺寸的φ分位数（0≤φ≤1））



## 向量匹配

向量之间的运算会尝试在左侧向量中为左侧的每个条目找到匹配的元素。匹配行为有两种基本类型：一对一和多对一/一对多。

### 一对一向量匹配

**一对一地**从操作的每一侧找到一对唯一的条目。在默认情况下，这是遵循format的操作`vector1  vector2`。如果两个条目具有完全相同的一组标签和相应的值，则它们匹配。的`ignoring`关键字允许忽略匹配时某些标签，而 `on`关键字允许减少该组被认为标签来提供的列表中：

```
<vector expr> <bin-op> ignoring(<label list>) <vector expr>
<vector expr> <bin-op> on(<label list>) <vector expr>
```

输入示例：

```
method_code:http_errors:rate5m{method="get", code="500"}  24
method_code:http_errors:rate5m{method="get", code="404"}  30
method_code:http_errors:rate5m{method="put", code="501"}  3
method_code:http_errors:rate5m{method="post", code="500"} 6
method_code:http_errors:rate5m{method="post", code="404"} 21

method:http_requests:rate5m{method="get"}  600
method:http_requests:rate5m{method="del"}  34
method:http_requests:rate5m{method="post"} 120
```

查询示例：

```
method_code:http_errors:rate5m{code="500"} / ignoring(code) method:http_requests:rate5m
```

这将返回一个结果向量，其中包含最近5分钟内对每种方法的状态请求为500的HTTP请求的比例。没有`ignoring(code)`这些指标，就不会有匹配项，因为指标不会共享同一组标签。用方法的条目`put`，并`del`没有匹配，并且在结果不会显示出来：

```
{method="get"}  0.04            //  24 / 600
{method="post"} 0.05            //   6 / 120
```

### 多对一和一对多向量匹配

**多对一**和**一对多**匹配是指“一个”侧上的每个矢量元素都可以与“许多”侧上的多个元素匹配的情况。必须使用`group_left`或`group_right`修饰符明确要求此操作，其中左/右确定哪个向量具有更高的基数。

```
<vector expr> <bin-op> ignoring(<label list>) group_left(<label list>) <vector expr>
<vector expr> <bin-op> ignoring(<label list>) group_right(<label list>) <vector expr>
<vector expr> <bin-op> on(<label list>) group_left(<label list>) <vector expr>
<vector expr> <bin-op> on(<label list>) group_right(<label list>) <vector expr>
```

组修饰符随附的标签列表包含“一”侧的其他标签，这些标签将包含在结果指标中。对于`on`标签只能出现在列表之一中。结果向量的每个时间序列都必须是唯一可识别的。

*分组修饰符只能用于 [比较](https://prometheus.io/docs/prometheus/latest/querying/operators/#comparison-binary-operators)和 [算术](https://prometheus.io/docs/prometheus/latest/querying/operators/#arithmetic-binary-operators)。默认情况下`and`，as `unless`和 `or`操作与正确向量中的所有可能条目匹配。*

查询示例：

```
method_code:http_errors:rate5m / ignoring(code) group_left method:http_requests:rate5m
```

在这种情况下，左向量每个`method`标签值包含一个以上的条目。因此，我们使用来指示这一点`group_left`。现在，右侧的元素与`method`左侧带有相同标签的多个元素匹配：

```
{method="get", code="500"}  0.04            //  24 / 600
{method="get", code="404"}  0.05            //  30 / 600
{method="post", code="500"} 0.05            //   6 / 120
{method="post", code="404"} 0.175           //  21 / 120
```

*多对一和一对多匹配是应仔细考虑的高级用例。通常，正确使用会产生`ignoring()`所需的结果。*



## 二进制运算符优先级

下表列出了Prometheus中二进制运算符的优先级，从最高到最低。

1. `^`
2. `*`, `/`, `%`
3. `+`, `-`
4. `==`, `!=`, `<=`, `<`, `>=`, `>`
5. `and`, `unless`
6. `or`

优先级相同的运算符是左关联的。例如， `2 * 3 % 2`等效于`(2 * 3) % 2`。但是`^`是正确的关联，所以`2 ^ 3 ^ 2`等效于`2 ^ (3 ^ 2)`。



---



## 函数

关于 Prometheus 提供的一些函数，可以在： https://prometheus.io/docs/prometheus/latest/querying/functions/ 查看。



---



## 例子

官方文档：https://prometheus.io/docs/prometheus/latest/querying/examples/

这里举几个特殊的。

### 子查询

```
rate(http_requests_total[5m])[30m:1m]
```

```
max_over_time(deriv(rate(distance_covered_total[5s])[30s:5s])[10m:])
```

### 使用操作符，函数

```
# 返回的结果中，只带 instance 这个 label
sum by (instance) (prometheus_http_requests_total)
# 不带 instance 这个 label
sum without (instance) (prometheus_http_requests_total)
```



