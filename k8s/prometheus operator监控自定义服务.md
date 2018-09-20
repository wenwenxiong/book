###pormetheus operator与kube-prometheus
prometheus operator管理kubernetes环境的prometheus的配置，管理prometheus和alertmanager集群生命周期。kube-prometheus是针对prometheus operator定制的prometheus等套件的运行于kubernetes的配置集合，这些套件用于监控运行之下kubernetes集群。

###pormetheus operator
pormetheus operator定义了以下四个CRDs（ custom resource definitions）
* Prometheus,定义了prometheus在kubernetes的部署。
* ServiceMonitor，定义了那些服务需要被prometheus监控，operator会自动根据ServiceMonitor的内容生成Prometheus scrape configuration配置。
* Alertmanager，定义了Alertmanager在kubernetes的部署。
* PrometheusRule，定义了预期的 Prometheus rule file，可以被包含alerting的prometheus加载和记录rules。

部署了kube-prometheus后，可以查看到上述四类资源类的创建。
```
[root@k8snode1 ~]# kubectl get Alertmanager --all-namespaces
NAMESPACE    NAME      AGE
monitoring   mon       1h
[root@k8snode1 ~]# kubectl get PrometheusRule --all-namespaces
NAMESPACE    NAME                                                                 AGE
monitoring   mon-alertmanager-alertmanager.rules                                  1d
monitoring   mon-exporter-kube-controller-manager-kube-controller-manager.rules   1d
monitoring   mon-exporter-kube-etcd-etcd3.rules                                   1d
monitoring   mon-exporter-kube-scheduler-kube-scheduler.rules                     1d
monitoring   mon-exporter-kube-state-kube-state-metrics.rules                     1d
monitoring   mon-exporter-kubelets-kubelet.rules                                  1d
monitoring   mon-exporter-kubernetes-kubernetes.rules                             1d
monitoring   mon-exporter-node-node.rules                                         1d
monitoring   mon-kube-prometheus-general.rules                                    1d
monitoring   mon-prometheus-rules-prometheus.rules                                1d
[root@k8snode1 ~]# kubectl get Prometheus --all-namespaces
NAMESPACE    NAME             AGE
monitoring   mon-prometheus   1h
[root@k8snode1 ~]# kubectl get ServiceMonitor --all-namespaces
NAMESPACE    NAME                                   AGE
monitoring   mon-alertmanager                       1h
monitoring   mon-exporter-kube-controller-manager   1h
monitoring   mon-exporter-kube-dns                  1h
monitoring   mon-exporter-kube-etcd                 1h
monitoring   mon-exporter-kube-scheduler            1h
monitoring   mon-exporter-kube-state                1h
monitoring   mon-exporter-kubelets                  1h
monitoring   mon-exporter-kubernetes                1h
monitoring   mon-exporter-node                      1h
monitoring   mon-grafana                            1h
monitoring   mon-prometheus                         1h
monitoring   openebs-volume                         1h
monitoring   prometheus-operator                    1d
[root@k8snode1 ~]#
```
注：上面名为openebs-volume的ServiceMonitor不是kube-prometheus自带的，其余的都是。openebs-volume的ServiceMonitor是我为了在已部署好的prometheus上监控openebs存储卷而编写的。接下来的内容就是以此为例来说明如何定义ServiceMonitor使用kube-prometheus来监控kubernetes内部和外部服务。
###ServiceMonitor监控kubernetes服务
servicemonitor的内容定义帮助文档参考

https://github.com/coreos/prometheus-operator/blob/master/Documentation/api.md#servicemonitorspec

https://github.com/coreos/prometheus-operator/blob/master/Documentation/api.md#endpoint

https://github.com/coreos/prometheus-operator/blob/master/Documentation/api.md#relabelconfig

https://github.com/coreos/prometheus-operator/blob/master/example/prometheus-operator-crd/servicemonitor.crd.yaml#L82:19

servicemonitor里最重要的配置是endpoints，它配置成联系到kubernetes里的service对应的Endpoints。是对监控目标对象的定位。endpoints是通过namespaceSelector和selector来从kubernetes过滤寻找的。
在endpoint下最重要的是配置port和path，port指定暴露metrics的端口，path指定收集路径（默认为/metrics）。
下面以```openebs-servicemonitor.yaml```内容为例说明，详细的请参考上述网站文档。
```
[root@k8snode1 ~]# cat openebs-servicemonitor.yaml 
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: openebs-volume
  namespace: monitoring
  labels:
    app: openebs-volume
spec:
  namespaceSelector:
    any: true
  selector:
    matchLabels:
      openebs/controller-service: jiva-controller-service
  targetLabels:
      - vsm
  endpoints:
  - port: http
#    metricRelabelings:
#      - sourceLabels: [__meta_kubernetes_pod_label_vsm]
#        targetLabel: openebs_pv
#        regex: (.*)
#        replacement: ${1}
#        action: replace
```
1、为servicemonitor设置labels，kube-prometheus通过labels过滤寻找到servicemonitor，这里配置名为openebs-volume的servicemonitor的labels为app: openebs-volume。修改kube-promethues创建的promethues配置。使用命令
```
kubectl edit prometheus mon-prometheus -n monitoring
```
在以下内容增加一行openebs-volume。
```
serviceMonitorSelector:
    matchExpressions:
    - key: app
      operator: In
      values:
      - alertmanager
      - exporter-coredns
      - exporter-kube-controller-manager
      - exporter-kube-dns
      - exporter-kube-etcd
      - exporter-kube-scheduler
      - exporter-kube-state
      - exporter-kubelets
      - exporter-kubernetes
      - exporter-node
      - grafana
      - prometheus
      - prometheus-operator
      - openebs-volume
```
2、namespaceSelector配置为任意namespaces中寻找，selector配置labels具有```openebs/controller-service: jiva-controller-service```的endpoints。

3、找到的endpoints里名为http的port为metrics收集端口，采用默认路径```/metrics```。

4、targetLabels把endpoints自带的label加入到收集指标的元数据标识中，这里是我自加的，因为我环境的目标endpoints每个都带有vsm标签。在helm安装的kube-prometheus里默认没有开启此功能，需要修改chart包里面的prometheus下的servicemonitors.yaml模板文件（具体修改参考网址https://github.com/coreos/prometheus-operator/issues/1604）。

5、在openebs创建的volume中默认没有把9500（就是http端口）暴露到service中（也就不会在endpoints显示），所以需要手动修改加入，目前还没有找到哪里可以配置自动加入9500端口。

###ServiceMonitor监控kubernetes外部服务
参考网址：https://devops.college/prometheus-operator-how-to-monitor-an-external-service-3cb6ac8d5acb

看完上述，读者大概清楚了servicemonitor如何连接kube-prometheus和kubernetes里的service。没错就是通过endpoints。并且使用labels来过滤和需找目标Endpoints。因此，可以为外部服务创建kubernetes里的service和Endpoint，然后配置kube-prometheus的查找label来指向它，便可以收集外部服务的监控指标，从而监控外部服务。
例如，创建Endpoint为
```
apiVersion: v1
kind: Endpoints
metadata:
    name: gpu-metrics
    labels:
        k8s-app: gpu-metrics
subsets:
    - addresses:
    - ip: <gpu-machine-ip>
ports:
    - name: metrics
      port: 9100
      protocol: TCP
```
创建service
```
apiVersion: v1
kind: Service
metadata:
    name: gpu-metrics-svc
    namespace: monitoring
    labels:
        k8s-app: gpu-metrics
spec:
    type: ExternalName
    externalName: <gpu-machine-ip>
    clusterIP: ""
    ports:
    - name: metrics
      port: 9100
      protocol: TCP
      targetPort: 9100
```
创建servicemonitor
```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
    name: gpu-metrics-sm
    labels:
        k8s-app: gpu-metrics
        prometheus: kube-prometheus
spec:
    selector:
        matchLabels:
            k8s-app: gpu-metrics
        namespaceSelector:
            matchNames:
            - monitoring
    endpoints:
    - port: metrics
      interval: 10s
      honorLabels: true
```
这里需要配置kube-prometheus需找label为```prometheus: kube-prometheus```的servicemonitor即可收集测试的外部服务指标。