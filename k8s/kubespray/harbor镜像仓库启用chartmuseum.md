###harbor启用chartmuseum

部署```harbor```时，添加参数```--with-chartmuseum```。
```
./prepare && ./install.sh --with-chartmuseum
```
修改```docker-infra.service```文件，加上``` -f /usr/local/compose/docker-compose.chartmuseum.yml```。
```
[Unit]
Description=Docker infra
Requires=docker.service
After=docker.service

[Service]
WorkingDirectory=/usr/local/compose/
Type=idle
Restart=always
# Remove old container items
ExecStartPre=/usr/bin/docker-compose -f /usr/local/compose/docker-compose.yml -f /usr/local/compose/docker-compose.chartmuseum.yml down
# Compose up
ExecStart=/usr/bin/docker-compose -f /usr/local/compose/docker-compose.yml -f /usr/local/compose/docker-compose.chartmuseum.yml up
# Compose stop
ExecStop=/usr/bin/docker-compose -f /usr/local/compose/docker-compose.yml -f /usr/local/compose/docker-compose.chartmuseum.yml stop

[Install]
WantedBy=multi-user.target
```
修改```/usr/local/compose/docker-compose.yml```,加入
```
...
services:
  chartmuseum:
    networks:
      harbor:
        aliases:
          - chartmuseum
...
```
在配置文件```inventory/rong/group_vars/k8s-cluster/k8s-cluster.yml```加入配置项
```
...
helm_stable_repo_url: "https://portus.teligen.com:5000/chartrepo/kubesprayns"
...
```
###上传下载charts
使用```helm```上传```charts```文件
首先安装```helm``插件```push```。下载```https://github.com/chartmuseum/helm-push/releases``` 的源码和二进制包。
```
helm-push-0.7.1.tar.gz
helm-push_0.7.1_linux_amd64.tar.gz
```
解压源码包，修改脚本文件```helm-push-0.7.1/scripts/install_plugin.sh```。
```
...
elif [ "$(uname)" = "Linux" ] ; then
    url="http://192.168.122.1/helm-push_${version}_linux_amd64.tar.gz"
...
```
上传```helm-push_0.7.1_linux_amd64.tar.gz```二进制包到```http```服务器```192.168.122.1```。执行安装命令
```
[root@node1 ~]# helm plugin install /root/helm-push-0.7.1
Downloading and installing helm-push v0.7.1 ...
http://192.168.122.1/helm-push_0.7.1_linux_amd64.tar.gz
Installed plugin: push
[root@node1 ~]# 
[root@node1 ~]# 
[root@node1 ~]# helm plugin list
NAME    VERSION DESCRIPTION                      
push    0.7.1   Push chart package to ChartMuseum
[root@node1 ~]#
```
注，```/root/helm-push-0.7.1```是解压并修改脚本后的源码包。

上传```chart```包命令
```
[root@node1 ~]# helm repo list
NAME    URL                                                  
stable  https://portus.teligen.com:5000/chartrepo/kubesprayns
local   http://127.0.0.1:8879/charts                         
[root@node1 ~]# 
[root@node1 ~]# helm push --username kubespray --password Thinker@1 rook-ceph-v0.9.0-4.g8a49531.tgz stable
Pushing rook-ceph-v0.9.0-4.g8a49531.tgz to stable...
Done.
[root@node1 ~]#
```
下载```charts```
```
[root@node1 ~]# helm search stable
NAME                    CHART VERSION           APP VERSION     DESCRIPTION                                                 
stable/rook-ceph        v0.9.0-4.g8a49531                       File, Block, and Object Storage Services for your Cloud-N...
stable/traefik          1.55.1                  1.7.4           A Traefik based Kubernetes ingress controller with Let's ...
[root@node1 ~]# helm fetch stable/rook-ceph --version v0.9.0-4.g8a49531

```
安装```charts```。
```
helm install stable/rook-ceph --version v0.9.0-4.g8a49531
```