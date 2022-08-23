# 深入探索Prometheus Exporter

## 前言

当下开源发展超乎想象，连腾讯，阿里这种大厂都在不断追赶各种开源，享受开源的红利。 开源时代的到来颠覆了传统IT生命周期发展的步伐，自研时代的“重开发”，到开源时代的“重集成”，这种“重集成”的就需要强有力的集成运维支撑。开源软件不像企业开发的商业闭源软件，商业软件一般是自完备的，从功能到安全到性能到监控都能自我掌控，开源软件往往在核心功能外的领域不够完备，往往需要企业自己增强。尤其在监控领域比较缺失，而Prometheus正好弥补这个空间，提供一种灵活方便的监控告警解决方案。

## Prometheus vs Zabbix

说到监控，几乎没人没听过Zabbix，Zabbix的成熟度自然非常高，而且配置UI相当强大， 但是Prometheus产生于容器时代，具有云原生和时序数据库的特点， 直接弯道超车，渐有取代Zabbix之势。

看下面网上整理的两者的区别：

![img](https:////upload-images.jianshu.io/upload_images/18097588-147d644adb325c78.png?imageMogr2/auto-orient/strip|imageView2/2/w/847/format/webp)

图片.png

### Zabbix

Zabbix核心组件主要是Agent和Server，其中Agent主要负责采集数据并通过主动或者被动的方式采集数据发送到Server/Proxy，Zabbix由于使用了关系型数据存储时序数据，所以在监控大规模集群时常常在数据存储方面捉襟见肘。所以从Zabbix 4.2版本后开始支持TimescaleDB时序数据库，不过目前成熟度还不高。

![img](https:////upload-images.jianshu.io/upload_images/18097588-c13d3ee141ef4ecb.png?imageMogr2/auto-orient/strip|imageView2/2/w/560/format/webp)

图片.png

### Prometheus

Prometheus的基本原理是通过HTTP周期性抓取被监控组件的状态，任意组件只要提供对应的HTTP接口并且符合Prometheus定义的数据格式，就可以接入Prometheus监控。

![img](https:////upload-images.jianshu.io/upload_images/18097588-4759bc069224ae7a.png?imageMogr2/auto-orient/strip|imageView2/2/w/560/format/webp)

图片.png

Prometheus Server负责定时在目标上抓取metrics（指标）数据并保存到本地存储里面。Prometheus采用了一种Pull（拉）的方式获取数据，不仅降低客户端的复杂度，客户端只需要采集数据，无需了解服务端情况，而且服务端可以更加方便的水平扩展。Prometheus自研一套高性能的tsdb时序数据库，在V3版本可以达到每秒千万级别的数据存储，通过对接第三方时序数据库扩展历史数据的存储。

![img](https:////upload-images.jianshu.io/upload_images/18097588-592a9b00ba5928c2.png?imageMogr2/auto-orient/strip|imageView2/2/w/560/format/webp)

图片.png

Prometheus的本地存储为官方自研高性能时序数据库。

Prometheus的远端存储可以对接OpenTSDB、InfluxDB、Elasticsearch、M3db、Kafka等，其中M3db是目前非常受欢迎的后端存储，适用于大量历史监控数据的存储和查询，。

## Exporter接口

Prometheus通过exporter机制将监控metric数据收集到服务端，exporter是一个http（不支持https）服务端口，由Prometheus server定时拉取，如下示意：

![img](https:////upload-images.jianshu.io/upload_images/18097588-35239050dbf622f2.png?imageMogr2/auto-orient/strip|imageView2/2/w/560/format/webp)

图片.png

对http返回格式的要求见[官方文档](https://links.jianshu.com/go?to=https%3A%2F%2Fprometheus.io%2Fdocs%2Finstrumenting%2Fexposition_formats%2F%23text-based-format)。

### Exporter格式



```go
metric_name ["{" label_name "=" `"` label_value `"` { "," label_name "=" `"` label_value `"` } [ "," ] "}"] value [ timestamp ]
```

metric_name:指标的名称

label：指标维度的标签，可以由多个

value：指标具体的值

### Exporter示例



```bash
# HELP http_requests_total The total number of HTTP requests.

# TYPE http_requests_total counter

http_requests_total{method="post",code="200"} 1027 1395066363000

http_requests_total{method="post",code="400"} 3 1395066363000

# Escaping in label values:

msdos_file_access_time_seconds{path="C:\\DIR\\FILE.TXT",error="Cannot find file:\n\"FILE.TXT\""} 1.458255915e9

# Minimalistic line:

metric_without_timestamp_and_labels 12.47

# A weird metric from before the epoch:

something_weird{problem="division by zero"} +Inf -3982045

# A histogram, which has a pretty complex representation in the text format:

# HELP http_request_duration_seconds A histogram of the request duration.

# TYPE http_request_duration_seconds histogram

http_request_duration_seconds_bucket{le="0.05"} 24054

http_request_duration_seconds_bucket{le="0.1"} 33444

http_request_duration_seconds_bucket{le="0.2"} 100392

http_request_duration_seconds_bucket{le="0.5"} 129389

http_request_duration_seconds_bucket{le="1"} 133988

http_request_duration_seconds_bucket{le="+Inf"} 144320

http_request_duration_seconds_sum 53423

http_request_duration_seconds_count 144320

# Finally a summary, which has a complex representation, too:

# HELP rpc_duration_seconds A summary of the RPC duration in seconds.

# TYPE rpc_duration_seconds summary

rpc_duration_seconds{quantile="0.01"} 3102

rpc_duration_seconds{quantile="0.05"} 3272

rpc_duration_seconds{quantile="0.5"} 4773

rpc_duration_seconds{quantile="0.9"} 9001

rpc_duration_seconds{quantile="0.99"} 76656

rpc_duration_seconds_sum 1.7560473e+07

rpc_duration_seconds_count 2693
```

value：指标具体的值

### 四种指标

#### Counter：只增不减的累加指标

Counter就是一个计数器，表示一种累积型指标，该指标只能单调递增或在重新启动时重置为零，例如，您可以使用计数器来表示所服务的请求数，已完成的任务或错误。

#### Gauge：可增可减的测量指标

Gauge是最简单的度量类型，只有一个简单的返回值，可增可减，也可以set为指定的值。所以Gauge通常用于反映当前状态，比如当前温度或当前内存使用情况；当然也可以用于“可增加可减少”的计数指标。

#### Histogram：自带buckets区间用于统计分布的直方图

Histogram主要用于在设定的分布范围内(Buckets)记录大小或者次数。

例如http请求响应时间：0-100ms、100-200ms、200-300ms、>300ms 的分布情况，Histogram会自动创建3个指标，分别为：

事件发送的总次数<basename>_count：比如当前一共发生了2次http请求

所有事件产生值的大小的总和<basename>_sum：比如发生的2次http请求总的响应时间为150ms

事件产生的值分布在bucket中的次数<basename>_bucket{le="上限"}：比如响应时间0-100ms的请求1次，100-200ms的请求1次，其他的0次

### Summary：数据分布统计图

Summary和Histogram类似，都可以统计事件发生的次数或者大小，以及其分布情况。

Summary和Histogram都提供了对于事件的计数_count以及值的汇总_sum，因此使用_count,和_sum时间序列可以计算出相同的内容。

同时Summary和Histogram都可以计算和统计样本的分布情况，比如中位数，n分位数等等。不同在于Histogram可以通过histogram_quantile函数在服务器端计算分位数。 而Sumamry的分位数则是直接在客户端进行定义。因此对于分位数的计算。 Summary在通过PromQL进行查询时有更好的性能表现，而Histogram则会消耗更多的资源。相对的对于客户端而言Histogram消耗的资源更少。

## Exporter选型

Prometheus官方本身提供了很多专业的Exporter，同时各个厂家为Prometheus开发的Exporter也非常多，选择这些现成的Exporter自然是首选。可以打开[官方网站](https://links.jianshu.com/go?to=https%3A%2F%2Fprometheus.io%2Fdocs%2Finstrumenting%2Fexporters%2F)看大概由150多种Exporter，包含如下领域：

数据库

操作系统

持续集成平台

消息中间件

分布式存储系统

Http服务器

日志系统

几乎涵盖了目前流行的大部分系统，足以见得Prometheus在监控界的地位。每一个Exporter启动侯都是一个http服务，可以已进程的形式启动，也可以以Docker容器的方式启动。

## Exporter开发

### Exporter SDK

对于开源的平台，我们一般能从官方列表中找到对应的Exporter，继续我们的拿来主义就行，稍加集成就可以顺利接入。对于闭源或者自研的系统，提供一个Prometheus Exporter形式监控接口则会提升软件本身的监控能力。[官方](https://links.jianshu.com/go?to=https%3A%2F%2Fprometheus.io%2Fdocs%2Finstrumenting%2Fclientlibs%2F)提供了四种语言（Go/Java/Python/Rubby）的正式客户端库用来开发一个集成http server的Exporter库。非正式的也有一些其它语言的库。

![img](https:////upload-images.jianshu.io/upload_images/18097588-688cae84b40440b9.png?imageMogr2/auto-orient/strip|imageView2/2/w/558/format/webp)

图片.png

Exporter库基本实现了上图的功能，提供了一个HttpServer，实现GET /metrics接口，这个接口关联到一个Handle访问CollectorRegistry获取所有的指标信息，序列化为Text文本格式返回。其中ColloecorRetistry需要提前注册需要获取的各种指标对象。

由于Exporter本身提供了一个REST服务器，所以不会特别轻量，会带来一些线程的消耗，如果在一个机器上启动过多Exporter实例则需要注意，想办法减少Exporter实例的数量。

自研的Exporter的优势是可以在代码中埋点，精确代码中的监控业务数据。同时写好一个Exporter还有一些原则，可以[参考网页](https://www.jianshu.com/p/8fe293744ebe)。比如要统计http请求的次数，某一种错误产生的次数，当前active连接的个数等等。

### 扩展Node Exporter

[Node Exporter](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fprometheus%2Fnode_exporter)是Prometheus官方发布的用来监控主机资源信息的Exporter，这个Exporter本身提供了一个Textfile Collector对外接口，可以用来把一些可以脚本化的监控数据带上去，通过--collector.textfile.directory 参数指定本地文本收集路径:

/opt/exporter/node_exporter/node_exporter --collector.textfile.directory=/opt/exporter/node_exporter/key

### 自研Exporter

除了以上2种方式，也可以按照[规范](https://links.jianshu.com/go?to=https%3A%2F%2Fprometheus.io%2Fdocs%2Finstrumenting%2Fexposition_formats%2F%23text-based-format)自己开发Exporter服务，实现一个返回文本指标格式的HTTP Server，

比如一个用[Python Flask写的Exporter](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.cnblogs.com%2Fliujiliang%2Fp%2F9694685.html)。

## Prometheus Roadmap

l **服务器端支持指标元数据**  目前指标的数据类型和其它元数据只在客户端使用，没有在服务器端持久化和使用。后续计划在服务器端使用这些元数据，第一步是在内存中聚合这些数据并且通过API对外提供。

l **适配OpenMetrics\**\**格式指标**

l **回填时间序列**  回填需要加载大批量的过期数据，并且从其它监控系统传输旧数据

l **服务端支持SSL\**\**和认证** 目前Prometheus服务端，AlertManager和官方Exporter都没有使用SSL加密，并且没有认证机制，需要加入SSL和认证来保证传输安全。

l **支持生态系统** 目前Prometheus支持了一些客户端库和exporter，还有好多其它语言需要加入支持。



作者：老陕西
链接：https://www.jianshu.com/p/dc4dbb497559
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
