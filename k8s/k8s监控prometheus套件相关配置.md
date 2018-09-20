###监控时间周期配置
参考网址: https://prometheus.io/docs/prometheus/latest/storage/
prometheus套件中通过命令行参数
```
--storage.tsdb.path: 配置prometheus收集数据存放的存储路径。
--storage.tsdb.retention: 配置超过多久的数据可以被删除，默认为15天。
```
平均, Prometheus 收集一个sample数据使用大概 1-2 bytes，可以使用以下公式计算Prometheus server的所需存储容量:
```
needed_disk_space = retention_time_seconds * ingested_samples_per_second * bytes_per_sample
```
在prometheus套件chart包的values.yaml中设置prometheus.retention的值来配置，默认为24h。