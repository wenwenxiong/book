参考网址：https://akomljen.com/get-kubernetes-logs-with-efk-stack-in-5-minutes/
https://akomljen.com/get-kubernetes-cluster-metrics-with-prometheus-in-5-minutes/

本实验环境kubernetes使用prometheus加grafana来作监控，efk作为日志管理。
prometheus使用Prometheus operator和Helm chart kube-prometheus来部署。
efk使用Elasticsearch Operator和EFK chart来部署。
###上传chart到chartmuseum
日志组件的目录如下
```
[root@rhel74 logging]# pwd
/root/logging
[root@rhel74 logging]# ls
efk-0.1.0.tgz  elasticsearch-operator-0.1.5.tgz  logging
```
efk-0.1.0.tgz  elasticsearch-operator-0.1.5.tgz为两个char包，logging文件夹包含chart所需的image。
监控组件的目录如下
```
[root@rhel74 monitor]# pwd
/root/monitor
[root@rhel74 monitor]# ls
custom-values.yaml  kube-prometheus-0.0.83.tgz  monitor  prometheus-operator-0.0.26.tgz
```
kube-prometheus-0.0.83.tgz，prometheus-operator-0.0.26.tgz为两个chart包，monitor文件夹包含chart所需的image。
custom-values.yaml文件包含对prometheus存储等的修改变量值
```
[root@rhel74 monitor]# cat custom-values.yaml 
global:
  rbacEnable: true

alertmanager:
  storageSpec:
    volumeClaimTemplate:
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 10Gi

prometheus:
  storageSpec:
    volumeClaimTemplate:
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 10Gi

grafana:
  auth:
    anonymous:
      enabled: "false"
  adminPassword: "YourPass123#"
  ingress:
    enabled: false
    annotations:
      kubernetes.io/ingress.class: nginx
      kubernetes.io/tls-acme: "true"
    hosts: 
      - grafana.test.akomljen.com
    tls:
      - secretName: grafana-tls
        hosts:
          - grafana.test.akomljen.com
  storageSpec:
    accessMode: "ReadWriteOnce"
    resources:
      requests:
        storage: 10Gi
```
上面定义了存储的大小，禁用grafana的anonymous登录，设置grafana的admin用户密码。（这里没有使用ingress）
上传所以的chart包到chartmuseum
```
curl -L --data-binary "@efk-0.1.0.tgz" http://portus.teligen.com:8988/api/charts
curl -L --data-binary "@elasticsearch-operator-0.1.5.tgz" http://portus.teligen.com:8988/api/charts
curl -L --data-binary "@kube-prometheus-0.0.83.tgz" http://portus.teligen.com:8988/api/charts
curl -L --data-binary "@prometheus-operator-0.0.26.tgz" http://portus.teligen.com:8988/api/charts
```
###helm安装监控和日志组件
运行helm命令安装监控和日志组件
```
helm repo update
helm install --name es-operator --namespace logging  chartmuseum/elasticsearch-operator
helm install --name efk --namespace logging chartmuseum/efk

helm install --name prometheus-operator --namespace monitoring chartmuseum/prometheus-operator
helm install --name mon --namespace monitoring -f custom-values.yaml chartmuseum/kube-prometheus
```