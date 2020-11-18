1、本地存储

Prometheus 2.x 采用自定义的存储格式将样本数据保存在本地磁盘当中。如下所示，按照两个小时为一个时间窗口，将两小时内产生的数据存储在一个块(Block)中，每一个块中包含该时间窗口内的所有样本数据(chunks)，元数据文件(meta.json)以及索引文件(index)。
从失败中恢复
如果本地存储由于某些原因出现了错误，最直接的方式就是停止Prometheus并且删除data目录中的所有记录。当然也可以尝试删除那些发生错误的块目录，不过相应的用户会丢失该块中保存的大概两个小时的监控记录。

2、远程存储

```Prometheus```的本地存储设计可以减少其自身运维和管理的复杂度，同时能够满足大部分用户监控规模的需求。但是本地存储也意味着```Prometheus```无法持久化数据，无法存储大量历史数据，同时也无法灵活扩展和迁移。
为了保持```Prometheus```的简单性，```Prometheus```并没有尝试在自身中解决以上问题，而是通过定义两个标准接口```(remote_write/remote_read)```，让用户可以基于这两个接口对接将数据保存到任意第三方的存储服务中，这种方式在```Promthues```中称为```Remote Storage```。

方案：
1）使用Influxdb作为```Remote Storage（https://yunlzheng.gitbook.io/prometheus-book/part-ii-prometheus-jin-jie/readmd/prometheus-remote-storage）```。
2）使用clickhouse作为```Remote Storage（https://segmentfault.com/a/1190000015710814）```。