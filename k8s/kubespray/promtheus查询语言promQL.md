1、直接使用指标值查找数据

使用关键字```node_load1```可以查询出Prometheus采集到的主机负载的样本数据。

2、prometheus监控指标的类型

在```Prometheus```的存储实现上所有的监控样本都是以```time-series```的形式保存在```Prometheus```内存的```TSDB```（时序数据库）中，有些指标随着时间的变化这个指标返回的样本数据是在不断变化的，有些指标是一个持续增大的值，因为其反应的是累积使用时间。

为了能够帮助用户理解和区分这些不同监控指标之间的差异，```Prometheus```定义了4中不同的指标类型```(metric type)```：```Counter（计数器）、Gauge（仪表盘）、Histogram（直方图）、Summary（摘要）```。
在```Exporter```返回的样本数据中，其注释中也包含了该样本的类型。例如：
```
# HELP node_cpu Seconds the cpus spent in each mode.
# TYPE node_cpu counter
node_cpu{cpu="cpu0",mode="idle"} 362812.7890625
```

1）、```Counter：```只增不减的计数器
一般是累计的计数值。这样的指标可以使用```rate()```函数获取HTTP请求量的增长率：
```
rate(http_requests_total[5m])
```
可以使用```topk```函数查询当前系统中，访问量前10的```HTTP```地址：
```
topk(10, http_requests_total)
```

2）```Gauge：```可增可减的仪表盘
该类型指标侧重反映系统的当前状态，指标计数值可增可减。
可以通过```delta```函数获取样本在一段时间返回内的变化情况。
例如，计算CPU温度在两个小时内的差异：
```
delta(cpu_temp_celsius{host="zeus"}[2h])
```
还可以使用```deriv()```计算样本的线性回归模型，甚至是直接使用```predict_linear()```对数据的变化趋势进行预测。例如，预测系统磁盘空间在4个小时之后的剩余情况：
```
predict_linear(node_filesystem_free{job="node"}[1h], 4 * 3600)
```
3）使用Histogram和Summary分析数据分布情况
Prometheus定义Histogram和Summary的指标类型。Histogram和Summary主用用于统计和分析样本的分布情况。
通过Histogram和Summary类型的监控指标，我们可以快速了解监控样本的分布情况。
例如，指标```prometheus_tsdb_wal_fsync_duration_seconds```的指标类型为```Summary```。 它记录了```Prometheus Server中wal_fsync```处理的处理时间，通过访问```Prometheus Server```的```/metrics```地址，可以获取到以下监控样本数据：
```
# HELP prometheus_tsdb_wal_fsync_duration_seconds Duration of WAL fsync.
# TYPE prometheus_tsdb_wal_fsync_duration_seconds summary
prometheus_tsdb_wal_fsync_duration_seconds{quantile="0.5"} 0.012352463
prometheus_tsdb_wal_fsync_duration_seconds{quantile="0.9"} 0.014458005
prometheus_tsdb_wal_fsync_duration_seconds{quantile="0.99"} 0.017316173
prometheus_tsdb_wal_fsync_duration_seconds_sum 2.888716127000002
prometheus_tsdb_wal_fsync_duration_seconds_count 216
```
从上面的样本中可以得知当前```Prometheus Server```进行```wal_fsync```操作的总次数为216次，耗时2.888716127000002s。其中中位数```（quantile=0.5）```的耗时为0.012352463，9分位数（quantile=0.9）的耗时为0.014458005s。
在```Prometheus Server```自身返回的样本数据中，我们还能找到类型为```Histogram```的监控指标```prometheus_tsdb_compaction_chunk_range_bucket。```
```
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
与```Summary```类型的指标相似之处在于```Histogram```类型的样本同样会反应当前指标的记录的总数(以```_count```作为后缀)以及其值的总量（以```_sum```作为后缀）。不同在于```Histogram```指标直接反应了在不同区间内样本的个数，区间通过标签```len```进行定义。
同时对于```Histogram```的指标，我们还可以通过```histogram_quantile()```函数计算出其值的分位数。不同在于```Histogram```通过```histogram_quantile```函数是在服务器端计算的分位数。 而```Sumamry```的分位数则是直接在客户端计算完成。因此对于分位数的计算而言，```Summary```在通过```PromQL```进行查询时有更好的性能表现，而```Histogram```则会消耗更多的资源。反之对于客户端而言```Histogram```消耗的资源更少。在选择这两种方式时用户应该按照自己的实际场景进行选择。

###PromQL
1、完全匹配
PromQL支持使用```=和!=```两种完全匹配模式：
通过使用```label=value```可以选择那些标签满足表达式定义的时间序列；
反之使用```label!=value```则可以根据标签匹配排除时间序列；

2、正则匹配
除了使用完全匹配的方式对时间序列进行过滤以外，PromQL还可以支持使用正则表达式作为匹配条件，多个表达式之间使用|进行分离：
使用```label=~regx```表示选择那些标签符合正则表达式定义的时间序列；
反之使用```label!~regx```进行排除；
例如，如果想查询多个环节下的时间序列序列可以使用如下表达式：
```
http_requests_total{environment=~"staging|testing|development",method!="GET"}
```
3、范围查询
如果我们想过去一段时间范围内的样本数据时，我们则需要使用区间向量表达式。区间向量表达式和瞬时向量表达式之间的差异在于在区间向量表达式中我们需要定义时间选择的范围，时间范围通过时间范围选择器```[]```进行定义。例如，通过以下表达式可以选择最近5分钟内的所有样本数据：
```
http_request_total{}[5m]
```
该表达式将会返回查询到的时间序列中最近5分钟的所有样本数据。
除了使用m表示分钟以外，PromQL的时间范围选择器支持其它时间单位：
```
s - 秒
m - 分钟
h - 小时
d - 天
w - 周
y - 年
```
4、PromQL聚合操作
Prometheus提供了下列内置的聚合操作符
```
sum (求和)
min (最小值)
max (最大值)
avg (平均值)
stddev (标准差)
stdvar (标准差异)
count (计数)
count_values (对value进行计数)
bottomk (后n条时序)
topk (前n条时序)
quantile (分布统计)
```
5、prometheus内置函数
1）计算```Counter```指标的增长率
获得指标在2分钟内的平均增长率
```
increase(node_cpu[2m]) / 120
```
或者
```
rate(node_cpu[2m])
```
或者
```
irate(node_cpu[2m])
```
函数```irate```相比于```rate```函数提供了更高的灵敏度，不过当需要分析长期趋势或者在告警规则中，```irate```的这种灵敏度反而容易造成干扰。因此在长期趋势分析或者告警中更推荐使用```rate```函数。
2）预测```Gauge```指标变化趋势
函数```predict_linear```可以预测时间序列```v```在```t```秒后的值。它基于简单线性回归的方式，对时间窗口内的样本数据进行统计，从而可以对时间序列的变化趋势做出预测。例如，基于2小时的样本数据，来预测主机可用磁盘空间的是否在4个小时候被占满，可以使用如下表达式：
```
predict_linear(node_filesystem_free{job="node"}[2h], 4 * 3600) < 0
```