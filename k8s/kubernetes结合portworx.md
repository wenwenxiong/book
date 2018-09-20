参考网址：https://docs.portworx.com/scheduler/kubernetes/install.html
https://docs.portworx.com/scheduler/kubernetes/support.html
https://github.com/xiaoping378/k8s-deploy

软件版本
kubernetes:1.6.2

前提条件：运行portworx的host需保持时间同步。
环境：参照https://github.com/xiaoping378/k8s-deploy
已安装好的k8s集群

####1, kubernetses安装portworx

#####1) 运行etcd，准备未分区的硬盘/dec/sdb
```
$ export HostIP=192.168.124.226 #（本机ip）
$ docker run --net=host \
   -d --name etcd-v3.1.3 \
   --volume=/tmp/etcd-data:/etcd-data \
   quay.io/coreos/etcd:v3.1.3 \
   /usr/local/bin/etcd \
   --name my-etcd-1 \
   --data-dir /etcd-data \
   --listen-client-urls http://0.0.0.0:12379 \
   --advertise-client-urls http://${HostIP}:12379 \
   --listen-peer-urls http://0.0.0.0:12380 \
   --initial-advertise-peer-urls http://${HostIP}:12380 \
   --initial-cluster my-etcd-1=http://${HostIP}:12380 \
   --initial-cluster-token my-etcd-token \
   --initial-cluster-state new \
   --auto-compaction-retention 1
```

#####2）下载portworx的yml文件,运行portworx
```
$ curl -o px-spec.yaml "http://install.portworx.com?cluster=mycluster&master=true&kvdb=etcd://192.168.124.226:12379&drives=/dev/sdb"
$ kubectl apply -f px-spec.yaml
```

####2, kubernetes使用portworx卷

#####1）静态使用
```
$ /opt/pwx/bin/pxctl volume create testvol --size 2G #直接创建portworx volume
```
kubernetes通过在pod和PersistentVolume中定义
```
portworxVolume:
       volumeID: testvol
```
使用portworx volume

#####2）动态使用

定义storageclass
```
kind: StorageClass
 apiVersion: storage.k8s.io/v1beta1
 metadata:
   name: portworx-sc
 provisioner: kubernetes.io/portworx-volume
 parameters:
   repl: "1"
```
->定义PersistentVolumeClaim
```
kind: PersistentVolumeClaim
 apiVersion: v1
 metadata:
   name: pvcsc001
   annotations:
     volume.beta.kubernetes.io/storage-class: portworx-sc
 spec:
   accessModes:
     - ReadWriteOnce
   resources:
     requests:
       storage: 2Gi
```
->pod中定义
```
persistentVolumeClaim:
           claimName: pvcsc001
```
使用定义PersistentVolumeClaim
####建议

tips1：portworx安装时会检测host上是否装有
```
kernel-headers-`uname -r`
kernel-devel-`uname -r`
```
没有安装，则会到portworx的官网去下载，为节省时间，建议手动下载并安装这两个文件。
tips2：kubernetes使用portworx volume后，发现删除不了，一直停留在Terminating 状态
通过命令journalctl -lfu kubelet查看日志发现，无法删除pod的unmount的volume目录
解决办法如下：
lsattr /var/lib/kubelet/pods/2176d3a8-984f-11e7-98c4-5254004b207b/volumes/kubernetes.io~portworx-volume
chattr -i /var/lib/kubelet/pods/2176d3a8-984f-11e7-98c4-5254004b207b/volumes/kubernetes.io~portworx-volume/pvc-21522191-984f-11e7-98c4-5254004b207b
