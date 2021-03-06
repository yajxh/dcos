## 概念与使用

### 数据模型

Prometheus本质上以时间序列存储所有的数据，时间序列是指属于相同度量和相同标记维度集合的时间戳值构成的数据流。除了存储的时间序列，Prometheus也可以将查询结果生成临时导出的时间序列。

#### 指标名称和标签

每个时间序列都由其**指标名称**和**一组键值对**（也称为**标签**）唯一标识。

**指标名称**用来指定所测量的系统的某项功能特性（例如，`http_requests_total` - 收到的HTTP请求的总数）。指标名称可以包含ASCII字母和数字，以及下划线和冒号。它必须匹配正则表达式：`[a-zA-Z_:][a-zA-Z0-9_:]*`。

标签是Prometheus的多维数据模型的基础：对于同一指标名称的任何给定的标签组合标识了该指标的特定维度的实例（例如：使用方法POST到`/api/tracks`处理程序的所有HTTP请求）。查询语言允许基于这些维度进行过滤和聚合。更改任何标签值（包括添加或删除标签）将创建新的时间系列。

标签名称可以包含ASCII字母，数字以及下划线。它们必须匹配正则表达式`[a-zA-Z_][a-zA-Z0-9_]*`。以`__`开头的标签名称保留供内部使用。

标签值可以包含任何Unicode字符。

**最佳实践**

**（1）指标名称**

指标名称应该具有与该指标所属的域相关的（单字）应用前缀。前缀有时被客户端库称为命名空间（namespace），前缀通常是应用程序名称本身。不过，某些时候度量指标更通用，如由客户端库发布的标准化的度量指标。示例：

* **prometheus**\_notifications\_total \(特定于Prometheus服务器\)

* **process**\_cpu\_seconds\_total \(许多客户端库公布的指标\)  
  s\)

* **http**\_request\_duration\_seconds \(针对HTTP请求\)


指标名称必须有一个且只有一个度量单位（即，不要将秒与微秒，或者秒与字节数等多个度量单位混合）

指标名称应使用基本单位（例如，秒、字节、米，而不是毫秒，兆字节，公里）。

指标名称应该有一个后缀以复数形式来描述度量单位。注意，累积计数除了度量单位（如果适用）之外，还要有总和作为后缀。

* http\_request\_duration\_**seconds**

* node\_memory\_usage\_**bytes**

* http\_requests\_**total** \(不带度量单位的累加计数\)

* process\_cpu\_**seconds\_total** \(带有度量单位的累加计数\)


在所有标签维度上，指标名称都应该表示同样逻辑的被测量物。

* 请求持续时间

* 数据传输字节数

* 资源瞬时使用百分比


作为经验法则，给定度量的所有维度上的sum\(\)或avg\(\)应该是有意义的（尽管不一定有用），否则，请将数据拆分为多个指标。例如，用一个度量指标表示不同队列的容量是好的实践，而用一个指标表示队列容量及当前队列中的元素数则是不好的。

**（2）标签**

使用标签来区分正在测量的事物的特征：

* `api_http_requests_total` - 可以用标签区分不同的HTTP请求类型：`type="create|update|delete"`

* `api_request_duration_seconds` - 可以用标签区分请求的不同处理阶段：`stage="extract|transform|load"`


不要将标签名称放在度量指标名称中，因为这会引入冗余并导致混乱。

**警告：**键值标签对的每个唯一组合表示一个新的时间序列，这会显著增加存储的数据量。不要使用标签来存储具有高基数（标签具有许多不同的值）的维度，例如用户ID，电子邮件地址或其他无界值集合。

#### 样本（Samples）

样本形成实际的时间序列数据。每个样本包括：

* 一个64位浮点值

* 精度为毫秒时间戳


#### 符号（Notation）

给定一个指标名称和一组标签，时间序列通常使用以下表示法标识：

```
<metric name>{<label name>=<label value>, ...}
```

例如，具有指标名称`api_http_requests_total`和标签`method=“POST”`和`handler =“/messages”`的时间系列可以写成这样：

```
api_http_requests_total{method="POST", handler="/messages"}
```

注意，这与[OpenTSDB](http://opentsdb.net/)使用的符号相同。

### 指标类型 {#metric-types}

Prometheus的客户端库提供了四种核心的指标类型，这些类型目前只在客户端库和对接协议中区分，服务器端并没有使用这些类型信息而是将所有数据扁平化为无类型的时间序列。

#### Counter

Counter是一个累积度量指标，表示一个不断上升的数值，典型的用例是服务请求数，任务完成数，错误数等。如果表示度量指标的值相比会减小，不能使用Counter，而应使用下面的Gauge。

#### Gauge

Gauge是表示可以任意上下的单个数值的度量指标，典型的用例是温度，当前内存使用量等。

#### Histogram

Histogram对监测样本进行采样，并在配置的区间内对这些样本进行计数，并提供所有监测样本的值的总和。

#### Summary

类似于Histogram，Summary也对监测样本进行采样。尽管它也提供监测样本的总数和监测值的总和，它还在滑动时间窗口上按可配置的分位数（quantiles）进行统计。

Histogram和Summary的区别参考[此处](https://prometheus.io/docs/practices/histograms/)。

### 任务和实例

在Prometheus的世界中，任何单独抓取的目标称为**实例\(instance\)**，通常对应于单个进程。相同类型的实例的集合（为了可伸缩性或可靠性而复制）被称为**作业\(job\)**。

下述是监控一个API服务器的任务，该任务有四个复制实例：

* job: api-server
  * instance 1: 1.2.3.4:5670
  * instance 2: 1.2.3.4:5671
  * instance 3: 5.6.7.8:5670
  * instance 4: 5.6.7.8:5671


当Prometheus采集目标指标时，它会自动将一些标签附加到抓取的时间序列中，用于识别被抓取的目标：

* `job`：抓取目标关联的已配置的作业名称

* `instance`：抓取的目标地址的`<host>:<port>`部分


如果上述两个标签中的任何一个在已抓取的数据中已存在，如何处理取决于`honor_labels`配置选项。

对于每一次instance的抓取操作，Prometheus按照以下时间序列存储样本：

* `up{job="<job-name>", instance="<instance-id>"}`: 1，如果该instance正常，即正常抓取；0，如果抓取失败。

* `scrape_duration_seconds{job="<job-name>", instance="<instance-id>"}`: 抓取的持续时间。


`up`时间序列对于实例可用性监控是有用的。

