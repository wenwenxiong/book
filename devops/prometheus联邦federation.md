## 简介

`Prometheus Federation` 允许一台 `Prometheus Server` 从另一台 `Prometheus Server` 刮取选定的时间序列资料。对于`Prometheus` 的 `Federation` 有不同的使用方式，一般分为`Cross-service federation`与`Hierarchical federation`。

`Cross-service federation`

![01](./prometheus/01.png)

`Hierarchical federation`

![02](./prometheus/02.png)

## 配置

在一个`prometheus`的配置文件`prometheus.yml`写入以下内容。

```
scrape_configs:
  - job_name: 'federate'
    scrape_interval: 15s

    honor_labels: true
    metrics_path: '/federate'

    params:
      'match[]':
        - '{job="prometheus"}'
        - '{__name__=~"job:.*"}'
        - '{job=~"prometheus.*"}'
        - '{job="docker"}'
        - '{job="node"}'
    static_configs:
      - targets:
        - 'source-prometheus-1:9090'
        - 'source-prometheus-2:9090'
        - 'source-prometheus-3:9090'
```

- 当设置 `Federation` 時，將通过 `params` 中的 `macth[]` 参数指定需要刮取的时间序列`job`，`match[]` 必须是一個`job`选择器，如 `up`或者 `{job="api-server"}` 等。
- 设定`honor_labels`是避免监控指标冲突。
- `targets`下指定目标`prometheus`。
