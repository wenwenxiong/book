## 原理

1、利用```netdata```的命令参数```--collector.textfile.directory```配置从指定文件读取监控指标信息。

2、利用```linux cron```定时任务脚本收集指定目录存储空间大小信息写入```--collector.textfile.directory```指定目录下的文件，供```netdata```读取。

## linux cron 定时任务脚本

编写文件```directory_size```，内容如下（示例中每5分钟收集一次文件目录```/var/log /usr/local/npg /home```，可以自己定义目录名称和数量，时间间隔）

```
*/5 * * * * root du -sb /var/log /usr/local/npg /home | sed -ne 's/^\([0-9]\+\)\t\(.*\)$/node_directory_size_bytes{directory="\2"} \1/p' > /usr/local/npg/textfile_collector/directory_size.prom.$$ && mv /usr/local/npg/textfile_collector/directory_size.prom.$$ /usr/local/npg/textfile_collector/directory_size.prom
```

把文件```directory_size```放入linux定时任务目录```/etc/cron.d```。

```
cp directory_size /etc/cron.d/
```

## 配置npg

监控告警系统```npg```使用```docker-compose```管理运行，对```docker-compose.yml```文件中的```nodeexporter```服务进行配置。

首先配置```node-exporter```启动命令参数加入```--collector.textfile.directory=/textfile_collector```。

其次配置```node-exporter```挂载主机文件目录```/usr/local/npg/textfile_collector```到容器内部目录```/textfile_collector```。

配置完成后，最后进行```npg```重启（npg master重启npg，worker节点只重启node-exporter服务）

## 附录

在```npg master```和```worker```上都运行```node-exporter```，因此都需要进行linux cron 定时任务脚本和node-exporter配置。