可以使用命令查看集群组件的健康状态
```
[root@node1 ~]# kubectl get cs
NAME                 STATUS    MESSAGE              ERROR
controller-manager   Healthy   ok                   
scheduler            Healthy   ok                   
etcd-0               Healthy   {"health": "true"}
```

查看kube-apiserver的健康检测获取错误信息
```
curl -v --cacert /etc/kubernetes/pki/ca.pem --key /etc/kubernetes/pki/admin-key.pem --cert /etc/kubernetes/pki/admin.pem  https://127.0.0.1:6443/healthz
```
如果有错误信息出现，在查询具体的```url```路径。
可以查的路径有
```
{
  "paths": [
    "/apis",
    "/apis/",
    "/apis/apiextensions.k8s.io",
    "/apis/apiextensions.k8s.io/v1beta1",
    "/healthz",
    "/healthz/etcd",
    "/healthz/ping",
    "/healthz/poststarthook/generic-apiserver-start-informers",
    "/healthz/poststarthook/start-apiextensions-controllers",
    "/healthz/poststarthook/start-apiextensions-informers",
    "/metrics",
    "/openapi/v2",
    "/swagger-2.0.0.json",
    "/swagger-2.0.0.pb-v1",
    "/swagger-2.0.0.pb-v1.gz",
    "/swagger-ui/",
    "/swagger.json",
    "/swaggerapi",
    "/version"
  ]
```
与健康检测```healthz```相关的有
```
    "/healthz",
    "/healthz/etcd",
    "/healthz/ping",
    "/healthz/poststarthook/generic-apiserver-start-informers",
    "/healthz/poststarthook/start-apiextensions-controllers",
    "/healthz/poststarthook/start-apiextensions-informers",
```
例如，如果查询
```
curl -v --cacert /etc/kubernetes/pki/ca.pem --key /etc/kubernetes/pki/admin-key.pem --cert /etc/kubernetes/pki/admin.pem  https://127.0.0.1:6443/healthz
```
报错
```
[+]ping ok\n[-]etcd failed: reason withheld\n[+]poststarthook/generic-apiserver-start-informers ok\n[+]poststarthook/start-cluster-operator-apiserver-informers ok\nhealthz check failed\n"
```
可以看出是```kube-apiserver```访问```etcd```检测出现了问题。
使用
```
curl -v --cacert /etc/kubernetes/pki/ca.pem --key /etc/kubernetes/pki/admin-key.pem --cert /etc/kubernetes/pki/admin.pem  https://127.0.0.1:6443/healthz/etcd
```
进一步查看错误细节。
一般出现这种情况，可能是```kube-apiserver```无法访问```etcd```，可能```kube-apiserver```无法DNS解析```etcd```节点的```hostname```，可以修改```kube-apiserver```的访问```etcd```集群参数，把```hostname```替换为```ip```，看能否解决问题。不行在看看是不是```ssl```证书的问题。