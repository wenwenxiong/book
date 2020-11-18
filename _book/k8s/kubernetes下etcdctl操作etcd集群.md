参考网址： https://github.com/coreos/etcd/blob/master/Documentation/dev-guide/interacting_v3.md

kubernetes使用etcd集群来存储数据信息，etcd是一个分布式键值存储。通过ectdctl可以对etcd进行操作。
###etcdctl操作etcd
etcdctl除了对etcd中的键值数据进行增删查（改其实就是用新值覆盖旧值），还可以查看历史版本数据值和一段时期progress的所有值变化，还有一个特性就是可以创建一个租期（leases），把键值对绑定到一个有效租期上，当租期超时后，该键值对自动删除。
####write/read/delete key-value

etcdctl写key-value值
```
$ etcdctl put foo bar
OK
```
etcdctl读key-value值
```
$ etcdctl get foo
foo
bar
```
只打印值
```
$ etcdctl get foo --print-value-only
bar
```
etcdctl读key的历史版本值
假设值为
```
foo = bar         # revision = 2
foo1 = bar1       # revision = 3
foo = bar_new     # revision = 4
foo1 = bar1_new   # revision = 5
```
读revision 为4
```
$ etcdctl get --prefix --rev=4 foo # access the versions of keys at revision 4
foo
bar_new
foo1
bar1
```
读keys大于指定的key
假设数值为
```
a = 123
b = 456
z = 789
```
读取keys大于b
```
$ etcdctl get --from-key b
b
456
z
789
```
etcdctl删除指定key
```
$ etcdctl del foo
1 # one key is deleted
```
####watch key-value
watch单个键值对
```
$ etcdctl watch foo
# in another terminal: etcdctl put foo bar
PUT
foo
bar
```
watch一段时期progress
```
$ etcdctl watch -i
$ watch a
$ progress
progress notify: 1
# in another terminal: etcdctl put x 0
# in another terminal: etcdctl put y 1
$ progress
progress notify: 3
```
压缩key的历史版本
```
$ etcdctl compact 5
compacted revision 5

# any revisions before the compacted one are not accessible
$ etcdctl get --rev=4 foo
Error:  rpc error: code = 11 desc = etcdserver: mvcc: required revision has been compacted
```
#### key-value with leases
首先创建一个租期（leases），然后绑定键值对到该租期。
```
# grant a lease with 10 second TTL
$ etcdctl lease grant 10
lease 32695410dcc0ca06 granted with TTL(10s)

# attach key foo to lease 32695410dcc0ca06
$ etcdctl put --lease=32695410dcc0ca06 foo bar
OK
```
删除租期（leases）会把关联的键值对都删除
```
$ etcdctl lease grant 10
lease 32695410dcc0ca06 granted with TTL(10s)
$ etcdctl put --lease=32695410dcc0ca06 foo bar
OK
Here is the command to revoke the same lease:

$ etcdctl lease revoke 32695410dcc0ca06
lease 32695410dcc0ca06 revoked

$ etcdctl get foo
# empty response since foo is deleted due to lease revocation
```
当租期（leases）快过期时，可以刷新周期
```
$ etcdctl lease keep-alive 32695410dcc0ca06
lease 32695410dcc0ca06 keepalived with TTL(10)
lease 32695410dcc0ca06 keepalived with TTL(10)
lease 32695410dcc0ca06 keepalived with TTL(10)
```
获取租期（lease）信息
```
$ etcdctl lease timetolive 694d5765fc71500b
lease 694d5765fc71500b granted with TTL(500s), remaining(258s)

$ etcdctl lease timetolive --keys 694d5765fc71500b
lease 694d5765fc71500b granted with TTL(500s), remaining(132s), attached keys([zoo2 zoo1])

# if the lease has expired or does not exist it will give the below response:
Error:  etcdserver: requested lease not found
```
###kismatic k8s环境下使用etcdctl操作k8s的etcd存储
参考网址：https://jimmysong.io/kubernetes-handbook/guide/using-etcdctl-to-access-kubernetes-data.html
在kismatic k8s环境中可以通过以下命令访问etcd集群。
```
docker run --rm --net=host --env ETCDCTL_API=3 --volume=/etc/ssl/certs/:/etc/ssl/certs/:ro --volume=/etc/etcd_k8s:/etc/etcd_k8s:ro portus.teligen.com:5000/kismatic110/quay.io/coreos/etcd:v3.1.13 /usr/local/bin/etcdctl --endpoints='https://127.0.0.1:2379/' --cert=/etc/etcd_k8s/etcd-client.pem --key=/etc/etcd_k8s/etcd-client-key.pem --cacert=/etc/etcd_k8s/ca.pem get /registry/namespaces/default
```
我们使用kubectl命令获取的kubernetes的对象状态实际上是保存在etcd中的，使用下面的脚本可以获取etcd中的所有kubernetes对象的key
```
#!/bin/bash
# Get kubernetes keys from etcd
keys=`docker run --rm --net=host --env ETCDCTL_API=3 --volume=/etc/ssl/certs/:/etc/ssl/certs/:ro --volume=/etc/etcd_k8s:/etc/etcd_k8s:ro portus.teligen.com:5000/kismatic110/quay.io/coreos/etcd:v3.1.13 /usr/local/bin/etcdctl --endpoints='https://127.0.0.1:2379/' --cert=/etc/etcd_k8s/etcd-client.pem --key=/etc/etcd_k8s/etcd-client-key.pem --cacert=/etc/etcd_k8s/ca.pem get /registry --prefix -w json|python -m json.tool|grep key|cut -d ":" -f2|tr -d '"'|tr -d ","`
for x in $keys;do
  echo $x|base64 -d|sort
done
```
以下是
```
/registry/apiextensions.k8s.io/customresourcedefinitions/alertmanagers.monitoring.coreos.com
/registry/apiextensions.k8s.io/customresourcedefinitions/clusters.ceph.rook.io
/registry/apiextensions.k8s.io/customresourcedefinitions/prometheuses.monitoring.coreos.com
/registry/apiextensions.k8s.io/customresourcedefinitions/prometheusrules.monitoring.coreos.com
/registry/apiextensions.k8s.io/customresourcedefinitions/servicemonitors.monitoring.coreos.com
/registry/apiextensions.k8s.io/customresourcedefinitions/volumesnapshotdatas.volumesnapshot.external-storage.k8s.io
/registry/apiextensions.k8s.io/customresourcedefinitions/volumesnapshots.volumesnapshot.external-storage.k8s.io
...
/registry/namespaces/monitoring
/registry/namespaces/rook-ceph
/registry/pods/kube-system/calico-kube-controllers-58647b5656-h7wrn
/registry/pods/kube-system/calico-node-r2k4f
...
/registry/serviceaccounts/monitoring/default
/registry/serviceaccounts/monitoring/prometheus-operator
/registry/services/endpoints/default/kubernetes
/registry/services/endpoints/kube-system/heapster
...
```
可以看到所有的Kuberentes的所有元数据都保存在/registry目录下，下一层就是API对象类型（复数形式），再下一层是namespace，最后一层是对象的名字。
