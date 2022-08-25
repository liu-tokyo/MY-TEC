# Prometheus

本文对Prometheus进行学习，参考官网文档[Overview](https://link.zhihu.com/?target=https%3A//prometheus.io/docs/introduction/overview/), [Promethus Book](https://link.zhihu.com/?target=https%3A//yunlzheng.gitbook.io/prometheus-book/)。

监控分为**白盒监控**和**黑盒监控**。通过白盒能够了解其内部的实际运行状态，通过对监控指标的观察能够预判可能出现的问题，从而对潜在的不确定因素进行优化。而黑盒监控，常见的如HTTP探针，TCP探针等，可以在系统或者服务在发生故障时能够快速通知相关的人员进行处理。

[Prometheus](https://link.zhihu.com/?target=https%3A//github.com/prometheus) 是开源的system monitoring and alerting toolkit，是在Kubernetes之后第二个加入CNCF( [Cloud Native Computing Foundation](https://link.zhihu.com/?target=https%3A//cncf.io/) )的项目。

## **Feature**

- 多维的data model来展示具有时间序列的数据
- 提供PromQL查询语言
- 不依赖分布式数据存储，单个server node是自治的
- 时间序列的collection提供HTTP的pull mode来获取
- 通过中间网关可以实现push model的时间序列
- 目标通过服务发现或静态配置来找到
- 支持多种模式的graph和dashboard

## **Component**

Prometheus ecosystem包含多个组件，大部分是可选的。大部分的component使用Go语言编写。

- Prometheus server：用来获取和存储时间序列数据
- client library: 用来实现application code
- push gateway: 用来支持short-live的job
- exportors: 用来支持services like HAProxy, StatsD, Graphite
- alertmanager: 用来支持alert

## **Architecture**

Prometheus 使用配置的jobs来直接或间接的获取metrics，将samples保存在本地并在data上执行rules来aggregate及record新的时间序列数据并生成alert。Grafana或其他的API consumer可以抽象化收集的数据。

对于微服务，Prometheus能够提供多维的数据收集和查询。同时，可以辅助出现outage时快速分析和定位问题。每个Prometheus server都是独立的，不依赖于网络存储或其他remote service。

![img](https://pic4.zhimg.com/80/v2-ddb9f52cade064103dc7916d8f27a497_720w.jpg)

Prometheus Server为核心部件，负责实现对监控数据的获取，存储以及查询。可以通过静态配置管理监控目标，也可以配合使用Service Discovery的方式动态管理监控目标。从监控目标采集数据，按照时间序列的方式存储在本地磁盘，对外提供自定义的PromQL语言，实现对数据的查询以及分析。同时，可以通过联邦集群以及功能分区的方式对Prometheus Server进行扩展。

Exporter将监控数据采集的端点通过HTTP服务的形式暴露给Prometheus Server，使其通过Exporter提供的Endpoin**t端点获得监控数据。根据是否支持Prometheus监控，Exporter分为直接采集(Kubernetes，Etcd，Gokit)和**间接采集(Client Library编写该监控目标的监控采集程序)。

## **Install**

安装prometheus可以使用二进制包安装和Docker安装。

- 二进制包安装

  我们可以在 [Download](https://link.zhihu.com/?target=https%3A//prometheus.io/download/) 中下载相应OS的安装包，解压后，找到相应的程序prometheus (prometheus.exe for windows).

  执行下面命令查看命令语法。

  ```text
  ./prometheus --help
  ```

- Docker安装

  直接使用Prometheus image便可以。

  ```powershell
  docker pull prom/prometheus
  docker run -p 9090:9090 -v /etc/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
  ```

- 配置

  Prometheus的配置在文件prometheus.yml中，下面是配置文件的内容示例。详细的配置参见文章 [Configuration](https://link.zhihu.com/?target=https%3A//prometheus.io/docs/prometheus/latest/configuration/configuration/).

  ```yaml
  # my global config
  global:
    scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
    evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
    # scrape_timeout is set to the global default (10s).
  
  # Alertmanager configuration
  alerting:
    alertmanagers:
    - static_configs:
      - targets:
        # - alertmanager:9093
  
  # Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
  rule_files:
    # - "first_rules.yml"
    # - "second_rules.yml"
  
  # A scrape configuration containing exactly one endpoint to scrape:
  # Here it's Prometheus itself.
  scrape_configs:
    # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
    - job_name: 'prometheus'
  
      # metrics_path defaults to '/metrics'
      # scheme defaults to 'http'.
  
      static_configs:
      - targets: ['localhost:9090']
  ```

  global中控制prometheus的全局配置，scrape_internal设置全局的从targets获取数据的频率，特定target上的设置会覆盖全局设置，evaluation_internal设置evaluate rules的频率。Prometheus使用rules来生成时间序列数据并发出alert。

  rule_files指定加载rule的位置。scrape_configs指定监控的resource。Prometheus也通过HTTP endpoint来暴露自己的数据，这样就能监控自己的health。默认只有一个job prometheus，用来获取Prometheus server暴露的时间序列数据。target的数据通过 /metrics endpoint来暴露，因此，默认job通过 http://localhost:9090/metrics 来获取数据。

  Promtheus作为一个时间序列数据库，其采集的数据会以文件的形似存储在本地中，默认的存储路径为./data，也可以通过参数`--storage.tsdb.path="data/"`来修改本地数据存储的路径。

- 启动

  下面使用指定的配置文件来启动prometheus。

  ```powershell
  ./prometheus --config.file=prometheus.yml
  ```

  ![img](https://pic3.zhimg.com/80/v2-9343d69d792e53a6024af45d7ddbe9ae_720w.jpg)

  启动之后，便可以访问 [http://localhost:9090](https://link.zhihu.com/?target=http%3A//localhost%3A9090/) 来查看状态。也可以通过 http://localhost:9090/metrics 来查看prometheus自己的metrics。

- 查询

  Prometheus具有内置的expression browser。访问 http://localhost:9090/graph 并在graph tab下选择console。Prometheus自己暴露的metric的名字为promhttp_metric_handler_requests_total，当用其作为expression code时，可以获取该名字对应的所有metrics，但具有不同的`labels`，不同`labels`表示不同的状态。

  ![img](https://pic4.zhimg.com/80/v2-26d969597c17e7417316011d6d814cbb_720w.jpg)
  下面为`expression code`的示例，更多`expression language`的信息参见[Query](https://link.zhihu.com/?target=https%3A//prometheus.io/docs/prometheus/latest/querying/basics/)。

  ```text
  //只返回结果为HTTP code 200的requests
  promhttp_metric_handler_requests_total{code="200"} 
  
  //返回的时间序列数据的数量
  count(promhttp_metric_handler_requests_total)
  ```

  图像接口访问 [http://localhost:9090/graph](https://link.zhihu.com/?target=http%3A//localhost%3A9090/graph) 中的graph tab。下面的expression表示获取1分钟内`request result`为200的频率。

  ```text
  rate(promhttp_metric_handler_requests_total{code="200"}[1m])
  ```

  关于Prometheus和其他同类产品的比较，参见文章 [Comparation](https://link.zhihu.com/?target=https%3A//prometheus.io/docs/introduction/comparison/) 。

## **Exporter**

**Prometheus**架构中，`Prometheus Server`主要负责数据的收集，存储并且对外提供数据查询支持。为了能够监控到某些东西，如主机的CPU使用率，我们需要使用到Exporter。`Prometheus`周期性的从`Exporter`暴露的`HTTP`服务地址（通常是/metrics）拉取监控样本数据。

`Exporter`可以独立于监控目标，也可以内置于监控目标，只要能向`Prometheus`提供标准格式的监控样本数据即可。`Exporter`的一个实例称为`target`，如下所示，`Prometheus`通过轮询的方式定期从这些`target`中获取样本数据。

![img](https://pic4.zhimg.com/80/v2-10d02b7e7c2ce379e014568f47ea68fb_720w.jpg)

`Exporter`可以是社区提供的，也可以是用户自定义的。下面是社区提供的Exporter，可以实现大部分通用的监控功能。同时，用户可以根据`Prometheus`提供的`Client Library`创建自己的`Exporter`程序。

![img](https://pic2.zhimg.com/80/v2-2786874c851a7bc810f44171bef71de9_720w.jpg)

所有的`Exporter`程序都需要按照`Prometheus`的规范，返回监控的样本数据。`Node Exporter`中，访问/metrics地址时会返回以下内容：样本的一般注释信息HELP，样本的类型注释信息TYPE，样本。

```text
# HELP node_cpu Seconds the cpus spent in each mode. -> HELP <metrics_name> <doc_string>
# TYPE node_cpu counter   -> TYPE <metrics_name> <metrics_type>
node_cpu{cpu="cpu0",mode="idle"} 362812.7890625
# HELP node_load1 1m load average.
# TYPE node_load1 gauge
node_load1 3.0703125
```

下面是类型为`summary`和`histogram`的样例：

```text
# A histogram, which has a pretty complex representation in the text format:
# HELP http_request_duration_seconds A histogram of the request duration.
# TYPE http_request_duration_seconds histogram
http_request_duration_seconds_bucket{le="0.05"} 24054
http_request_duration_seconds_bucket{le="0.1"} 33444
http_request_duration_seconds_bucket{le="0.2"} 100392
http_request_duration_seconds_bucket{le="+Inf"} 144320
http_request_duration_seconds_sum 53423
http_request_duration_seconds_count 144320

# Finally a summary, which has a complex representation, too:
# HELP rpc_duration_seconds A summary of the RPC duration in seconds.
# TYPE rpc_duration_seconds summary
rpc_duration_seconds{quantile="0.01"} 3102
rpc_duration_seconds{quantile="0.05"} 3272
rpc_duration_seconds{quantile="0.5"} 4773
rpc_duration_seconds_sum 1.7560473e+07
rpc_duration_seconds_count 2693
```

可以使用cAdvisor来监控容器运行状态，详情参见文章 [cAdvisor](https://link.zhihu.com/?target=https%3A//yunlzheng.gitbook.io/prometheus-book/part-ii-prometheus-jin-jie/exporter/commonly-eporter-usage/use-prometheus-monitor-container) 。使用`MySQLD Exporter`来监控MySQL的性能，连接情况，使用情况等信息，详情参见文章 [MYSQLD](https://link.zhihu.com/?target=https%3A//yunlzheng.gitbook.io/prometheus-book/part-ii-prometheus-jin-jie/exporter/commonly-eporter-usage/use-promethues-monitor-mysql)。



- **Node Exporter**

  为了能够采集到主机的运行指标如CPU, 内存，磁盘等信息。我们可以使用 [Node Exporter](https://link.zhihu.com/?target=https%3A//github.com/prometheus/node_exporter) 。

  下载并执行Node Exporter:

  ```text
  curl -OL https://github.com/prometheus/node_exporter/releases/download/v0.18.1/node_exporter-0.18.1.linux-amd64.tar.gz
  tar -xzf node_exporter-0.18.1.linux-amd64.tar.gz
  cd node_exporter-0.18.1.linux-amd64
  cp node_exporter-0.18.1.linux-amd64/node_exporter /usr/local/bin/
  node_exporter
  ```
  
  启动成功后，可以看到输出：
  
  ```text
  transwarp@transwarp-Latitude-5490:~/下载/node_exporter-0.18.1.linux-amd64$ node_exporter 
  INFO[0000] Starting node_exporter (version=0.18.1, branch=HEAD, revision=3db77732e925c08f675d7404a8c46466b2ece83e)  source="node_exporter.go:156"
  INFO[0000] Build context (go=go1.12.5, user=root@b50852a1acba, date=20190604-16:41:18)  source="node_exporter.go:157"
  INFO[0000] Enabled collectors:                           source="node_exporter.go:97"
  INFO[0000]  - arp                                        source="node_exporter.go:104"
  INFO[0000]  - bcache                                     source="node_exporter.go:104"
  INFO[0000]  - bonding                                    source="node_exporter.go:104"
  INFO[0000]  - conntrack                                  source="node_exporter.go:104"
  INFO[0000]  - cpu                                        source="node_exporter.go:104"
  INFO[0000]  - cpufreq                                    source="node_exporter.go:104"
  INFO[0000]  - diskstats                                  source="node_exporter.go:104"
  INFO[0000]  - edac                                       source="node_exporter.go:104"
  INFO[0000]  - entropy                                    source="node_exporter.go:104"
  INFO[0000]  - filefd                                     source="node_exporter.go:104"
  INFO[0000]  - filesystem                                 source="node_exporter.go:104"
  INFO[0000]  - hwmon                                      source="node_exporter.go:104"
  INFO[0000]  - infiniband                                 source="node_exporter.go:104"
  INFO[0000]  - ipvs                                       source="node_exporter.go:104"
  INFO[0000]  - loadavg                                    source="node_exporter.go:104"
  INFO[0000]  - mdadm                                      source="node_exporter.go:104"
  INFO[0000]  - meminfo                                    source="node_exporter.go:104"
  INFO[0000]  - netclass                                   source="node_exporter.go:104"
  INFO[0000]  - netdev                                     source="node_exporter.go:104"
  INFO[0000]  - netstat                                    source="node_exporter.go:104"
  INFO[0000]  - nfs                                        source="node_exporter.go:104"
  INFO[0000]  - nfsd                                       source="node_exporter.go:104"
  INFO[0000]  - pressure                                   source="node_exporter.go:104"
  INFO[0000]  - sockstat                                   source="node_exporter.go:104"
  INFO[0000]  - stat                                       source="node_exporter.go:104"
  INFO[0000]  - textfile                                   source="node_exporter.go:104"
  INFO[0000]  - time                                       source="node_exporter.go:104"
  INFO[0000]  - timex                                      source="node_exporter.go:104"
  INFO[0000]  - uname                                      source="node_exporter.go:104"
  INFO[0000]  - vmstat                                     source="node_exporter.go:104"
  INFO[0000]  - xfs                                        source="node_exporter.go:104"
  INFO[0000]  - zfs                                        source="node_exporter.go:104"
  INFO[0000] Listening on :9100                            source="node_exporter.go:170"
  ```

  访问 [http://localhost:9100/](https://link.zhihu.com/?target=http%3A//localhost%3A9100/) 可以看到以下页面：

  ![img](https://pic1.zhimg.com/80/v2-fcf7131db87fb3617873a14b7f068720_720w.jpg)

  访问 http://localhost:9100/metrics 可以看到Node exportor获取到的当前主机的所有监控数据。所有监控metrics以下面形式展示，HELP解释metric，TYPE说明metric的类型。
  
  ```text
  # HELP node_cpu Seconds the cpus spent in each mode.
  # TYPE node_cpu counter
  node_cpu{cpu="cpu0",mode="idle"} 362812.7890625
  # HELP node_load1 1m load average.
  # TYPE node_load1 gauge
  node_load1 3.0703125
  ```

  为了Prometheus Server能够收集Node Exportor的数据，将相关信息放入`prometheus.yml`中并重启Prometheus Server。这里使用static config方式来静态配置instance，Prometheus还支持与DNS、Consul、E2C、Kubernetes等进行集成实现自动发现Instance实例，并从这些Instance上获取监控数据。
  
  ```yaml
  scrape_configs:
    - job_name: 'prometheus'
      static_configs:
        - targets: ['localhost:9090']
    # 采集node exporter监控数据
    - job_name: 'node'
      static_configs:
        - targets: ['localhost:9100']
  ```
  
  访问 http://localhost:9090 进入Prometheus server，输入up执行后便可以看到node job已经启动(1表示启动正常，0表示异常)。访问 http://localhost:9090/targets 可以直接从Prometheus的UI中查看当前所有的任务以及每个任务对应的实例信息。

  ![img](https://pic4.zhimg.com/80/v2-a522d23ab1faef47af80ed8ad8639ebb_720w.jpg)

  ![img](https://pic4.zhimg.com/80/v2-fa82e5dc740ec2bfe2a12830ae40bd37_720w.jpg)

## **可视化监控**

Grafana是一个开源的可视化平台，提供了对Prometheus的完整支持。执行下面的命令来启动Grafana，访问 http://localhost:3000 进入Grafana主界面，默认使用账号admin/admin。

```text
docker run -d -p 3000:3000 grafana/grafana
```

在Grafana的主界面中，可以添加数据源，Dashboard。

添加Dashboard,添加一个类型为Graph的面板，然后在其Metrics中通过PromQL中查询需要可视化的数据，保存后便创建了Dashboard。同时，可以通过网站 [https://grafana.com/dashboards](https://link.zhihu.com/?target=https%3A//grafana.com/dashboards) 找到分享的dashboard,这些dashboard通过JSON文件分享，下载并导入JSON文件便可以直接使用定义好的dashboard。

**Data Model**

Prometheus功能上将属于相同metric中具有相同label集合的所有data以时间序列存放。每个**time series**都可以通过metric name及其labels(key-value对)来唯一识别。metric name表示通用feature，label用来实现多维的data model。增删label或修改label value都将生成新的time series。

**Samples**构成了实际的time series data. 每个Sample包含一个float64 value以及一个微秒精度的timestamp。Prometheus会将所有采集到的样本数据以时间序列（time-series）的方式保存在内存数据库中，并且定时保存到硬盘上。

Time series通过**Notation**来识别，语法如下：

```powershell
<metric name>{<label name>=<label value>, ...}
样例：
api_http_requests_total{method="POST", handler="/messages"}
等价于
{__name__="api_http_requests_total"，method="POST", handler="/messages"}
```

### **Metric Type**

Prometheus client libraries提供了4种核心的metric type，但目前只在client libraries和wire protocol上有区别，Prometheus server并没有使用这些type info,而是将所有类型都当做untyped time series。将来可能会改变。

- Counter

  用作累计的metric,数值只能增加或在restart时被重置为0，使用场景：request数量，完成的任务数量或者发生的error数量。

  提供的Java API参见 [Java API](https://link.zhihu.com/?target=https%3A//github.com/prometheus/client_java/blob/master/simpleclient/src/main/java/io/prometheus/client/Counter.java).

- Gauge

  用作单个的数据值，可以任意增加或减少。使用场景：记录温度，CPU使用率，并发调用数。提供的Java API参见 [Java API](https://link.zhihu.com/?target=https%3A//github.com/prometheus/client_java/blob/master/simpleclient/src/main/java/io/prometheus/client/Gauge.java).

- Histogram

  表示样本观察者，如：request时长，response大小等，并在可配置的buckets中统计这些值，并提供对所有的观察值进行求和。统计在不同区间内样本的个数，区间通过标签len进行定义。

  Histogram同时暴露了多个time series：<basename>_bucket，<basename>_sum，<basename>_count。

  提供的Java API参见 [Java API](https://link.zhihu.com/?target=https%3A//github.com/prometheus/client_java/blob/master/simpleclient/src/main/java/io/prometheus/client/Histogram.java).
  
  ```text
  # HELP prometheus_tsdb_compaction_chunk_range Final time range of chunks on their first compaction
  # TYPE prometheus_tsdb_compaction_chunk_range histogram
  prometheus_tsdb_compaction_chunk_range_bucket{le="100"} 0
  prometheus_tsdb_compaction_chunk_range_bucket{le="400"} 0
  prometheus_tsdb_compaction_chunk_range_bucket{le="1600"} 0
  prometheus_tsdb_compaction_chunk_range_bucket{le="6400"} 0
  prometheus_tsdb_compaction_chunk_range_bucket{le="25600"} 0
  prometheus_tsdb_compaction_chunk_range_bucket{le="102400"} 0
  prometheus_tsdb_compaction_chunk_range_bucket{le="409600"} 0
  prometheus_tsdb_compaction_chunk_range_bucket{le="1.6384e+06"} 260
  prometheus_tsdb_compaction_chunk_range_bucket{le="6.5536e+06"} 780
  prometheus_tsdb_compaction_chunk_range_bucket{le="2.62144e+07"} 780
  prometheus_tsdb_compaction_chunk_range_bucket{le="+Inf"} 780
  prometheus_tsdb_compaction_chunk_range_sum 1.1540798e+09
  prometheus_tsdb_compaction_chunk_range_count 780
  ```

- Summary

  与Histogram类似，summary也表示样本观察，同时提供observation的总数以及所有被观察值的总和。记录在指定分位数对应的值。下面示例中，0.5分位的时间为0.0123，0.9分位的时间为0.0144.

  ```text
  # HELP prometheus_tsdb_wal_fsync_duration_seconds Duration of WAL fsync.
  # TYPE prometheus_tsdb_wal_fsync_duration_seconds summary
  prometheus_tsdb_wal_fsync_duration_seconds{quantile="0.5"} 0.012352463
  prometheus_tsdb_wal_fsync_duration_seconds{quantile="0.9"} 0.014458005
  prometheus_tsdb_wal_fsync_duration_seconds{quantile="0.99"} 0.017316173
  prometheus_tsdb_wal_fsync_duration_seconds_sum 2.888716127000002
  prometheus_tsdb_wal_fsync_duration_seconds_count 216
  ```

  提供的Java API参见 [Java API](https://link.zhihu.com/?target=https%3A//github.com/prometheus/client_java/blob/master/simpleclient/src/main/java/io/prometheus/client/Summary.java).

### **Jobs and Instances**

Prometheus中，一个用来获取数据的endpoint被称为instance，通常对应一个进程。具有相同purpose的一组instances被称为job。

下面的api server job具有4个instance：

![img](https://pic4.zhimg.com/80/v2-48ab2b2bd99b310e6e1aac52636aeda7_720w.jpg)

当要从target获取数据时，Prometheus会自动添加标签job和instance来形成time series，用来识别scraped target. 对每个instance，Prometheus会在下面的time series中添加sample。

- up{job="<job-name>", instance="<instance-id>"}: 1表示healthy， 0表示fail
- scrape_duration_seconds{job="<job-name>", instance="<instance-id>"}表示scrape的duration
- scrape_samples_post_metric_relabeling{job="<job-name>", instance="<instance-id>"}表示重打label后搜集的sample数量
- scrape_samples_scraped{job="<job-name>", instance="<instance-id>"}表示target暴露的sample数量
- scrape_series_added{job="<job-name>", instance="<instance-id>"}表示新series的数量

### **Get Started**

Prometheus通过target上的HTTP endpoints来从monitored targets上搜集metrics。默认情况下，Prometheus将database存放在./data文件夹下，也可以通过--storage.tsdb.path来指定。

Prometheus支持使用recording rules来prerecord expressions到新的time series中。下面是rule file `prometheus.rules.yml`：

```yaml
groups:
- name: example
  rules:
  - record: job_service:rpc_durations_seconds_count:avg_rate5m
    expr: avg(rate(rpc_durations_seconds_count[5m])) by (job, service)
```

然后将该rule file添加进prometheus.yml文件中，重启prometheus即可。

安装Prometheus的方法包括：二进制文件(前面安装的方式)，source文件，Docker方式。可以从 [Quay.io](https://link.zhihu.com/?target=https%3A//quay.io/repository/prometheus/prometheus) 或 [Docker Hub](https://link.zhihu.com/?target=https%3A//hub.docker.com/r/prom/prometheus/) 上获取Prometheus service的docker镜像。

获取并执行该镜像如下：

```bash
docker pull prom/prometheus
docker run -p 9090:9090 prom/prometheus
```

Prometheus image使用volume来存放actual metrics。

为了提供own configuration, 可以使用Volumes & bind-mount或custom image的方式。下面，我们使用custom image的方式，将configuration放入image中，避免在host上管理configuration file，适应于configuration比较static,在不同环境中基本相同的场景。

生成Dockerfile: 将prometheus.yml文件添加进docker的/etc/prometheus文件夹中。

```text
FROM prom/prometheus
ADD prometheus.yml /etc/prometheus/
```

然后，build和run新生成的image。

```text
docker build -t my-prometheus .
docker run -p 9090:9090 my-prometheus
```

### **Configuration**

Prometheus可以使用command line flags或configuration file来配置。前者通常用于不变的system parameters，后者定义关于job, instance, rule file的一些配置。Prometheus的配置可以runtime修改。

配置文件使用--config.file来指定，以YAML格式提供。下面提供了文件的样式，详细的配置请参见 [Configuration](https://link.zhihu.com/?target=https%3A//prometheus.io/docs/prometheus/latest/configuration/configuration/) 。

```yaml
global:
  # How frequently to scrape targets by default.
  [ scrape_interval: <duration> | default = 1m ]

  # How long until a scrape request times out.
  [ scrape_timeout: <duration> | default = 10s ]

  # How frequently to evaluate rules.
  [ evaluation_interval: <duration> | default = 1m ]

  # The labels to add to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
    [ <labelname>: <labelvalue> ... ]

# Rule files specifies a list of globs. Rules and alerts are read from
# all matching files.
rule_files:
  [ - <filepath_glob> ... ]

# A list of scrape configurations.
scrape_configs:
  [ - <scrape_config> ... ]

# Alerting specifies settings related to the Alertmanager.
alerting:
  alert_relabel_configs:
    [ - <relabel_config> ... ]
  alertmanagers:
    [ - <alertmanager_config> ... ]

# Settings related to the remote write feature.
remote_write:
  [ - <remote_write> ... ]

# Settings related to the remote read feature.
remote_read:
  [ - <remote_read> ... ]
```

### **Recording/Alerting Rules**

Prometheus支持2种类型的rule: recording rules和alerting rules。使用rule，需要创建rule file，其中包含rule statement并在配置文件中加载rule file。Rule file可以在运行时通过向prometheus process发送SIGHUP信号来重新加载。只有当rule file的格式正确时才会被使用。

在不启动Prometheus server的情况下检查rule file的语法是否正确，可以使用工具promtool。当语法正确时，程序返回0并rule的文本表示；否则返回1并打印错误信息。

```text
go get github.com/prometheus/prometheus/cmd/promtool
promtool check rules /path/to/example.rules.yml
```

Recording rule和alerting rule存在于rule group中，group中的rule以相同的间隔顺序执行。语法如下：

```yaml
groups:
  name: <string>  # group name, must be unique in a file
  [interval: <duration>] | default=global.evaluation_interval #evaluation frequency
  rules:
    #For recording rule
    record:<string> #the name of the time series to output to, must be valid metric name
    expr:<string> #The PromQL expression to evaluate
    labels:  #labels to add or overwrite bvefore storing the result
      [<labelname>: <labelvalue>]

    #For alerting rules
    alert:<string> #alert name, must be valid metric name
    expr:<string>  #The PromQL expression to evaluate
    [for: <duration> | default=0s]
    labels:
      [<labelname>: <labelvalue>]
    annotations:
      [<labelname>: <tmpl_string>]
```

Alerting rule使用Prometheus expression language来定义alert conditions并向external service发送alert。

### **Querying**

Prometheus提供了功能性查询语言PromQL，用户可以实时查询并aggregate time series。查询结果可以使用Prometheus expression browser来展示并可以被external system通过HTTP API来消费。

Expression Language的数据类型包括：Instant vector(一组time series，每个具有单个sample且具有相同的timestamp)； Range vector(一组time series，每个包含基于时间的data points)；scalar(浮点数)；string.

String常量使用单引号、双引号、反引号辅助表示，但是反引号中不支持转义字符。

```text
"this is a string"
'these are unescaped: \n \\ \t'
`these are not unescaped: \n ' " \t`
```

下面是Instant Vector的示例：

```text
http_requests_total
http_requests_total{job="prometheus",group="canary"}
```

第二条限制获取job名和group指定的time series。也可以反向限制选择的对象。注意：匹配空label的label matcher将会过滤出没有label的time series。匹配分为完全匹配和正则匹配。

![img](https://pic2.zhimg.com/80/v2-4df90773f7377a4e7542dd696a3000fd_720w.jpg)

Range vector跟instant vector类似，只是在当前时间为基准的情况下，根据设定的时间往前推迟获取time range之内的数据。duration放在[]中，跟在vector selector后面。

```text
http_requests_total{job="prometheus"}[5m]
```

offset modifier用来修改instant vector或range vector的time offset，在当前时间基准的基础上往前推进指定offset时间作为基准时间。时间单位有s,m,h,d,w,y.

```text
http_requests_total offset 5m   //5分钟前的瞬时样本数据
sum(http_requests_total{method="GET"} offset 5m) // GOOD.
rate(http_requests_total[5m] offset 1w)
```

下面是一些聚合操作：

```text
# 查询系统所有http请求的总量
sum(http_request_total)

# 按照mode计算主机CPU的平均使用时间
avg(node_cpu) by (mode)

# 按照主机查询各个主机的CPU使用率
sum(sum(irate(node_cpu{mode!='idle'}[5m]))  / sum(irate(node_cpu[5m]))) by (instance)
```

- Operator

算数运算符：+,-,*,/,%,^.可作用于scalar以及vector之间，作用于vector是对其中每个元素进行操作。

比较运算符：==,!=,>,<,>=,<=.默认进行filter，但在运算符后提供bool将返回0或1，而不是filter。作用于scalar之间时必须提供bool，返回0,1来表示比较结果。

```text
(node_memory_bytes_total - node_memory_free_bytes_total) / node_memory_bytes_total > 0.95  //filter
http_requests_total > bool 1000  //0 or 1
```

集合运算符：and(交集), or(并集), unless(complement).只能作用于instant vector。unless表示在v1中且不在v2中的元素组成的集合。

操作符优先级：

![img](https://pic2.zhimg.com/80/v2-2dd4d85e98bfd2ba30ff9b471eca32e1_720w.jpg)

匹配：

匹配分位一对一匹配，多对一匹配和一对多匹配。

一对一匹配要求两边标签必须完全一致，在操作符两边表达式标签不一致的情况下，可以使用on(label list)或者ignoring(label list）来修改便签的匹配行为。使用ignoreing可以在匹配时忽略某些便签。而on则用于将匹配行为限定在某些便签之内。

```text
<vector expr> <bin-op> ignoring(<label list>) <vector expr>
<vector expr> <bin-op> on(<label list>) <vector expr>
```

多对一和一对多两种匹配模式指的是“一”侧的每一个向量元素可以与"多"侧的多个元素匹配的情况。在这种情况下，必须使用group修饰符：group_left或者group_right来确定哪一个向量具有更高的基数（充当“多”的角色）。

```text
<vector expr> <bin-op> ignoring(<label list>) group_left(<label list>) <vector expr>
<vector expr> <bin-op> ignoring(<label list>) group_right(<label list>) <vector expr>
<vector expr> <bin-op> on(<label list>) group_left(<label list>) <vector expr>
<vector expr> <bin-op> on(<label list>) group_right(<label list>) <vector expr>
```

聚合操作符

作用于instant vector。

![img](https://pic1.zhimg.com/80/v2-5408dbb9f78deb1bd08e277ab952ce70_720w.jpg)

聚合操作语法如下：

```text
<aggr-op>([parameter,] <vector expression>) [without|by (<label list>)]
```

without用于从计算结果中移除列举的标签，而保留其它标签。by则正好相反，结果向量中只保留列出的标签，其余标签则移除。通过without和by可以按照样本的问题对数据进行聚合。

内置函数：

increase(v range-vector)函数是PromQL中提供的众多内置函数之一。其中参数v是一个区间向量，increase函数获取区间向量中的第一个和最后一个样本并返回其增长量。

```text
increase(node_cpu[2m]) / 120
```

rate函数可以直接计算区间向量v在时间窗口内平均增长速率。

```text
rate(node_cpu[2m])
```

irate同样用于计算区间向量的计算率，但是其反应出的是瞬时增长率。irate函数是通过区间向量中最后两个样本数据来计算区间向量的增长速率。这种方式可以避免在时间窗口范围内的“长尾问题”。

```text
irate(node_cpu[2m])
```

predict_linear函数可以预测时间序列v在t秒后的值。它基于简单线性回归的方式，对时间窗口内的样本数据进行统计，从而可以对时间序列的变化趋势做出预测。下面基于2小时的样本来预测未来4小时空间是否被占满。

```text
predict_linear(node_filesystem_free{job="node"}[2h], 4 * 3600) < 0
```

Histogram的分位数计算需要通过histogram_quantile(φ float, b instant-vector)函数进行计算。其中φ（0<φ<1）表示需要计算的分位数，如果需要计算中位数φ取值为0.5。

```text
histogram_quantile(0.5, http_request_duration_seconds_bucket)
```

为了能够让客户端的图标更具有可读性，可以通过label_replace标签为时间序列添加额外的标签。该函数会依次对v中的每一条时间序列进行处理，通过regex匹配src_label的值，并将匹配部分relacement写入到dst_label标签中。

```text
label_replace(v instant-vector, dst_label string, replacement string, src_label string, regex string)
```

Prometheus还提供了label_join函数，该函数可以将时间序列中v多个标签src_label的值，通过separator作为连接符写入到一个新的标签dst_label中:

```text
label_join(v instant-vector, dst_label string, separator string, src_label_1 string, src_label_2 string, ...)
```

HTTP API中使用promQL：

Prometheus当前稳定的HTTP API可以通过/api/v1访问，响应格式为：

```text
{
  "status": "success" | "error",
  "data": <data>,

  // Only set if status is "error". The data field may still hold
  // additional data.
  "errorType": "<string>",
  "error": "<string>"
}
```

通过HTTP API我们可以分别通过/api/v1/query和/api/v1/query_range查询PromQL表达式当前或者一定时间范围内的计算结果。

瞬间数据查询：

```text
GET /api/v1/query
```

URL请求参数：

- query=：PromQL表达式。
- time=：用于指定用于计算PromQL的时间戳。可选参数，默认情况下使用当前系统时间。
- timeout=：超时设置。可选参数，默认情况下使用-query,timeout的全局设置。

响应样例：

```text
$ curl 'http://localhost:9090/api/v1/query?query=up&time=2015-07-01T20:10:51.781Z'
{
   "status" : "success",
   "data" : {
      "resultType" : "vector",
      "result" : [
         {
            "metric" : {
               "__name__" : "up",
               "job" : "prometheus",
               "instance" : "localhost:9090"
            },
            "value": [ 1435781451.781, "1" ]
         },
         {
            "metric" : {
               "__name__" : "up",
               "job" : "node",
               "instance" : "localhost:9100"
            },
            "value" : [ 1435781451.781, "0" ]
         }
      ]
   }
}
```

区间数据查询：

```text
GET /api/v1/query_range
```

URL请求参数：

- query=: PromQL表达式。
- start=: 起始时间。
- end=: 结束时间。
- step=: 查询步长。
- timeout=: 超时设置。可选参数，默认情况下使用-query,timeout的全局设置。

============看到 [Operator](https://link.zhihu.com/?target=https%3A//prometheus.io/docs/prometheus/latest/querying/operators/)，后面继续。