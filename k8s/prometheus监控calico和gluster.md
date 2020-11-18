###prometheus监控calico

首先，在部署```k8s```的部署脚本```kubespray```中开启```calico```暴露```felix```监控指标。
编辑文件```roles/network_plugin/calico/defaults/main.yml```，修改配置项```calico_felix_prometheusmetricsenabled```为```true```。
如果是已部署的```calico```，修改其```daemonset```，配置环境变量监控```felix```为```true```。

配置```prometheus```监控```calico```节点的```9091```端口，下载```calico grafana dashboard```导入grfana。
下载地址为```https://grafana.com/grafana/dashboards/3244```

###prometheus监控gluster

编译项目```https://github.com/gluster/gluster-prometheus```得到```gluster-exporter```，参考```github```项目文档使用```systemd```启动与管理```gluster-exporter```服务。端口默认为```8080```,可以通过配置文件修改。
编译过程
```
mkdir -p $GOPATH/src/github.com/gluster
cd $GOPATH/src/github.com/gluster
git clone https://github.com/gluster/gluster-prometheus.git
cd gluster-prometheus
PREFIX=/usr make
PREFIX=/usr make install
```
中间遇到国内网路问题，需要配置代理下载```go```的依赖包。（配置```cow```和```http```代理）参考
```
https://segmentfault.com/a/1190000018610099
https://blog.csdn.net/ys5773477/article/details/73929161
```

配置```prometheus```监控```gluster-exporter```节点的```9091```端口，目前无法找到一个开源的```grafana dashboard```来展示```gluster```性能数据。

```glusterfs```的搭建通过```3rdparty```下的部署脚本进行部署。