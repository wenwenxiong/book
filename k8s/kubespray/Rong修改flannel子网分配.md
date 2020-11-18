### 1、`flannel`切换使用`etcd`配置

目前，已部署的`flannel`从`configmap`读取网络配置（`https://github.com/coreos/flannel/blob/master/Documentation/configuration.md`），根据已知`bug`（`https://github.com/coreos/flannel/issues/1328`）这种情况下修改`SubnetLen`不成功。因此需要`flannel`切换到读取`etcd`配置。

1）修改`/etc/kubernetes/cni-flannel.yml`

修改`flannel`的`daemonset`启动命令为

```
...
command: ["/opt/bin/flanneld", "--ip-masq", "--kube-subnet-mgr=false", "--etcd-cafile=/etc/kube-flannel/ca.pem", "--etcd-certfile=/etc/kube-flannel/admin-arm-a1.pem","--etcd-keyfile=/etc/kube-flannel/admin-arm-a1-key.pem", "--etcd-endpoints=https://192.192.190.163:2379/"]
...
```

注： `192.192.190.163`换成`Rong`环境的`etcd`集群中其中一个节点`ip`。

注：`etcd`证书存在于`etcd`节点的`/etc/ssl/etcd/ssl`目录，`ca.pem`是固定存在的，额外寻找一对证书，例如上面例子选择`admin-arm-a1.pem`和`admin-arm-a1-key.pem`。

2）修改`flannel`的`configmap`，加入`etcd`的认证证书

首先创建名为`etcd-key`的`configmap`保存`etcd`证书。

```
# cd /etc/ssl/etcd/ssl
# kubectl create configmap etcd-key -n kube-system --from-file=ca.pem --from-file=admin-arm-a1.pem --from-file=admin-arm-a1-key.pem
```

然后将`configmap`为`etcd-key`的`data`数据内容拷贝追加到`/etc/kubernetes/cni-flannel.yml`文件中`configmap`的`data`内容中。

```
---
kind: ConfigMap
...
data:
  cni-conf.json: |
   ...
  net-conf.json: |
   ...
  admin-arm-a1-key.pem: |
    -----BEGIN RSA PRIVATE KEY-----
    ...
  admin-arm-a1.pem: |
    -----BEGIN CERTIFICATE-----
    ...
  ca.pem: |
    -----BEGIN CERTIFICATE-----
    ...
...
```



### 2、配置`etcd`数据库

首先，`etcd`数据库集群中其中一个节点需要开启`v2`版本`api`支持，具体修改方法如下

在部署了`etcd`的节点上修改文件`/usr/local/bin/etcd`文件，增加`--enable-v2`内容。

```
...
/usr/local/bin/etcd \
--enable-v2 \
"$@"
```

重启`etcd`服务

```
systemctl restart etcd
```

修改`etcd`数据库，创建`flannel`的网络配置。

```
# export ETCDCTL_API=2 etcdctl --endpoints=https://127.0.0.1:2379/ --cert-file=/etc/ssl/etcd/ssl/admin-arm-a1.pem --key-file=/etc/ssl/etcd/ssl/admin-arm-a1-key.pem --ca-file=/etc/ssl/etcd/ssl/ca.pem mkdir /coreos.com
# export ETCDCTL_API=2 etcdctl --endpoints=https://127.0.0.1:2379/ --cert-file=/etc/ssl/etcd/ssl/admin-arm-a1.pem --key-file=/etc/ssl/etcd/ssl/admin-arm-a1-key.pem --ca-file=/etc/ssl/etcd/ssl/ca.pem mkdir /coreos.com/network
# export ETCDCTL_API=2 etcdctl --endpoints=https://127.0.0.1:2379/ --cert-file=/etc/ssl/etcd/ssl/admin-arm-a1.pem --key-file=/etc/ssl/etcd/ssl/admin-arm-a1-key.pem --ca-file=/etc/ssl/etcd/ssl/ca.pem set /coreos.com/network/config '{"Network":"10.233.64.0/18","SubnetLen": 25,"Backend":{"Type": "vxlan","VNI": 1,"Port": 8472}}'
```

注： 网段`10.233.64.0/18`必须与`Rong`安装时配置的`pod cidr`保持一致，`Rong`的`pod cidr`配置由文件`rong-var.yml`的配置项`kube_pods_subnet`指定。

`SubnetLen`根据实际环境配置。

### 3、重新部署`flannel`

在部署节点执行以下命令重新部署`flannel`。

```
kubectl delete -f /etc/kubernetes/cni-flannel.yml && kubectl create -f /etc/kubernetes/cni-flannel.yml
```

重启每台机器，使节点上的`pod`重启获取新的分配`ip`。