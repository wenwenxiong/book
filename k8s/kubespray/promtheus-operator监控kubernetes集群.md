参考网址： https://github.com/helm/charts/tree/master/stable/prometheus-operator

在```stable```的```chart repo```中，存在```prometheus-operator```项目，与```coreos```的```prometheus-operator```不同，该项目已经与```kube-prometheus```结合，因此，只需要部署该项目便可以监控整个```kubernetes```集群。

该项目默认启动时
```
helm install --name my-release stable/prometheus-operator
```
配置了部署```Prometheus```、```Alertmanager```、```Grafana```。
在```Exporters```模块配置中包含了```kubeApiServer```，```kubelet```，```kubeControllerManager```，```coreDns```，```kubeDns```，```kubeEtcd```，```kubeScheduler```，```kubeStateMetrics```，```nodeExporter```。
应用到```kubespray```部署的```kubernetes```环境时，出现了以下问题
1、模块```kubelet```的监控信息开放端口```10255```没有暴露，获取不了数据。
2、模块```kubeControllerManager```和```kubeScheduler```的监听地址为```127.0.0.1```,因此监控端口连接访问被拒绝。
3、模块```kubeEtcd```需要```https```认证配置，默认启动时没有配置。

经过调研，测试，解决如下
1、在```kubespray```的配置文件```./inventory/ai/group_vars/all/all.yml```的81左右，取消注释配置
```
## The read-only port for the Kubelet to serve on with no authentication/authorization. Uncomment to enable.
kube_read_only_port: 10255
```
开放```10255```端口。

2、在```kubespray```的配置文件```./roles/kubernetes/master/templates/kubeadm-config.v1alpha3.yaml.j2```中配置模块```kubeControllerManager```和```kubeScheduler```的```address```参数。
```
...
controllerManagerExtraArgs:
  address: 0.0.0.0
...
schedulerExtraArgs:
  address: 0.0.0.0
...
```
配置监听地址为```0.0.0.0```.
3、修改```prometheus-operator```的```value.yaml```配置文件。
```
...
kubeEtcd:
  enabled: true

  endpoints:
    - 192.168.122.61
    - 192.168.122.62
    - 192.168.122.63

  service:
    port: 2379
    targetPort: 2379
    selector:
      k8s-app: etcd-server

  serviceMonitor:
    scheme: https
    insecureSkipVerify: false
    serverName: localhost
    caFile: /etc/prometheus/secrets/etcd-client-cert/etcd-ca
    certFile: /etc/prometheus/secrets/etcd-client-cert/etcd-client
    keyFile: /etc/prometheus/secrets/etcd-client-cert/etcd-client-key
...
```
把```serviceMonitor```的配置改为```https```。（如果```etcd```集群部署在```kubernetes```中，注意```selector```的配置，要确保该配置能找到```etcd```集群的```service```，其他模块同理，因此，需要修改模块```kubeControllerManager```和```kubeScheduler```的```selector```配置为```component: xxx```）。
证书以```secret```形式放在与```prometheus```同一名字空间下
并且在```value.yaml```配置文件```prometheus```配置区域，加入以下配置
```
 secrets:
      - etcd-client-cert
```
创建名为```etcd-client-cert```的```secret```。
```
cp /etc/kubernetes/ssl/etcd/ca.pem etcd-ca
cp /etc/kubernetes/ssl/etcd/node-node1.pem etcd-client
cp /etc/kubernetes/ssl/etcd/node-node1-key.pem etcd-client-key
kubectl create secret generic etcd-client-cert --from-file=etcd-ca --from-file=etcd-client --from-file=etcd-client-key -n monitoring
```
至此，解决对```etcd```集群的```https```的访问。

此外，在查看```prometheus```的```Recording Rules```文件```./templates/alertmanager/rules/node.rules.yaml```时，发现其配置默认的网络接口设备名字为```eth0```。如果安装```kubernetes```环境的节点网络接口为其他名字，则会收集不到网络性能监控信息，需要更改为正确的网络接口设备名字。