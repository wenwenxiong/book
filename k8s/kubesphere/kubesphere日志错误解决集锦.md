# kube-state-metrics rbac权限问题
表现：在```kube-state-metrics pod```中持续报错

```
Failed to list *v1beta1.CertificateSigningRequest: certificatesigningrequests.certificates.k8s.io is forbidden: User "system:serviceaccount:kubesphere-monitoring-system:kube-state-metrics" cannot list resource "certificatesigningrequests" in API group "certificates.k8s.io" at the cluster scope
E0303 03:29:15.187251       1 reflector.go:125] k8s.io/kube-state-metrics/internal/store/builder.go:295: Failed to list *v1.StorageClass: storageclasses.storage.k8s.io is forbidden: User "system:serviceaccount:kubesphere-monitoring-system:kube-state-metrics" cannot list resource "storageclasses" in API group "storage.k8s.io" at the cluster scope

```
解决方案：在```kubesphere-kube-state-metrics clusterrole```中增加以下内容
```
- apiGroups:
  - storage.k8s.io
  resources:
  - storageclasses
  verbs:
  - list
  - watch
- apiGroups:
  - certificates.k8s.io
  resources:
  - certificatesigningrequests
  verbs:
  - list
  - watch
```

# fluent-big报错es请求排队阻塞

表现：```fluent-bit```有下面的报错

```
[2019/11/20 03:36:10] [error] [out_es] could not pack/validate JSON response
{“took”:23440,“errors”:true,“items”:[{“index”:{“index”:“ks-logstash-log-2019.08.31”,“type”:“flb_type”,“id”:“Amjhhm4BYm92d9lXXhZ7”,“status”:429,“error”:{“type”:“es_rejected_execution_exception”,“reason”:“rejected execution of processing of [1221794][indices:data/write/bulk[p]]: request: BulkShardRequest [[ks-logstash-log-2019.08.31][4]] containing [30] requests, target allocation id: pneqd10bS9OHDbGIt10PeA, primary term: 1 on EsThreadPoolExecutor[name = elasticsearch-logging-data-0/write, queue capacity = 200, org.elasticsearch.common.util.concurrent.EsThreadPoolExecutor@778891ba[Running, pool size = 4, active threads = 4, queued tasks = 200, completed tasks = 632026]]”}}},{“index”:{“index”:“ks-logstash-log-2019.08.31″,“type”:“flb_type”,“id”:“A2jhhm4BYm92d9lXXhZ7″,“status”:429,“error”:{“type”:“es_rejected_execution_exception”,“reason”:"rejected execution of processing of [1221792][indices:data/write/bulk[p]]: re
```
解决方案：
1）把es的resource limit去掉
2）扩容 elasticsearch-logging-data POD 的副本数到5个

# kube-state-metrics addon-resizer资源获取及更新失败

表现：```addon-resizer```容器报错
```
 Resources are not within the expected limits, updating the deployment. Actual: {Limits:map[] Requests:map[cpu:{i:{value:10 scale:-3} d:{Dec:<nil>} s:10m Format:DecimalSI} memory:{i:{value:157286400 scale:0} d:{Dec:<nil>} s:150Mi Format:BinarySI}]} Expected: {Limits:map[cpu:{i:{value:34000000 scale:-9} d:{Dec:<nil>} s: Format:DecimalSI} memory:{i:{value:0 scale:0} d:{Dec:0xc420450390} s: Format:BinarySI}] Requests:map[cpu:{i:{value:34000000 scale:-9} d:{Dec:<nil>} s: Format:DecimalSI} memory:{i:{value:0 scale:0} d:{Dec:0xc420450390} s: Format:BinarySI}]}
ERROR: logging before flag.Parse: E0303 01:59:48.706689       1 nanny_lib.go:110] the server could not find the requested resource
ERROR: logging before flag.Parse: I0303 02:00:01.422945       1 nanny_lib.go:108] Resources are not within the expected limits, updating the deployment. Actual: {Limits:map[] Requests:map[memory:{i:{value:157286400 scale:0} d:{Dec:<nil>} s:150Mi Format:BinarySI} cpu:{i:{value:10 scale:-3} d:{Dec:<nil>} s:10m Format:DecimalSI}]} Expected: {Limits:map[cpu:{i:{value:34000000 scale:-9} d:{Dec:<nil>} s: Format:DecimalSI} memory:{i:{value:0 scale:0} d:{Dec:0xc420459860} s: Format:BinarySI}] Requests:map[cpu:{i:{value:34000000 scale:-9} d:{Dec:<nil>} s: Format:DecimalSI} memory:{i:{value:0 scale:0} d:{Dec:0xc420459860} s: Format:BinarySI}]}
```
暂没有找出解决方案

# kube-dashboard报错日志每2秒刷一次，严重占用磁盘空间
表现：重启kubesphere集群后，有时会出现```kube-dashboard```报错如下

```
Restarting synchronizer: kubernetes-dashboard-key-holder-kube-system.
2020/03/03 03:38:14 Starting secret synchronizer for kubernetes-dashboard-key-holder in namespace kube-system
2020/03/03 03:38:14 Synchronizer kubernetes-dashboard-key-holder-kube-system exited with error: kubernetes-dashboard-key-holder-kube-system watch ended with timeout

```
临时解决方案： 重启```kube-dashboard pod```，日志不再出现，清空以前日志，释放空间。