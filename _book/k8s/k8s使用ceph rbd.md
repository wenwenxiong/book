环境
节点               | k8s角色       |      ceph 安装组件
------------------|---------------|------------------------------------
192.168.122.120   |   k8s-master  | ceph admin mon1 osd0 osd1 osd2 osd9
192.168.122.121   |   k8s-slave1  | ceph mon2 osd3 osd4 osd5 osd10
192.168.122.122   |   k8s-slave2  | ceph mon3 osd6 osd7 osd8 osd11

k8s使用kubeadmin部署，部署参考官方文档
https://kubernetes.io/docs/setup/independent/install-kubeadm/
https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/
kubernetes 1.7.5
ceph参考以下文档部署
http://www.xuxiaopang.com/2016/10/10/ceph-full-install-el7-jewel/
ceph version 10.2.2

####k8s以volume使用ceph rbd
使用官方提供的例子加以修改
```
apiVersion: v1
kind: Pod
metadata:
  name: rbd
spec:
  containers:
    - image: 192.168.122.1:5000/kubernetes/pause:v1
      name: rbd-rw
      volumeMounts:
      - name: rbdpd
        mountPath: /mnt/rbd
  volumes:
    - name: rbdpd
      rbd:
        monitors:
        - '192.168.122.120:6789'
        - '192.168.122.121:6789'
        - '192.168.122.122:6789'
        pool: ssd-pool
        image: rbdpd
        fsType: ext4
        readOnly: false
        user: admin
        keyring: /etc/ceph/ceph.client.admin.keyring
#        imageformat: "2"
#        imagefeatures: "layering"

```
这里无法定义使用size大小，估计rbd image多大，就是多大。
RBD可以设置readOnly: true可以被多个pod使用。但是设置readOnly: false则只能被一个pod使用。
可以将一个rbd image 设置readOnly: false被一个pod使用，写入数据。然后删除pod（rbd image仅仅被umount，数据还保留），然后将该rbd image设置readOnly: true可以被多个pod读取使用。

使用该yaml文件创建pod时，遇到不识别imageformat和imagefeatures关键字。
只有手动禁用rbd image的以下feature
```
rbd feature disable ssd-pool/rbdpd exclusive-lock object-map fast-diff deep-flatten
```
####k8s以pvc和pv组合使用ceph rbd
k8s volume还不能完全满足实际生产过程对持久化存储的需求，因为k8s volume的lifetime和pod的生命周期相同，一旦pod被delete，那么volume中的数据就不复存在了。于是k8s又推出了Persistent Volume(PV)和Persistent Volume Claim(PVC)组合，故名思意：即便挂载其的pod被delete了，PV依旧存在，PV上的数据依旧存在。

k8spod->pvc->pv
pvc定义过程需使用到storageclass，pv可以使用也可以不使用storageclass
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: rbdpv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: rbdsc
  rbd:
      monitors:
        - '192.168.122.120:6789,192.168.122.121:6789,192.168.122.122:6789'
      pool: ssd-pool
      image: rbdpd
      fsType: ext4
      readOnly: false
      user: admin
      keyring: /etc/ceph/ceph.client.admin.keyring

```
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: rbdpv8g
spec:
  capacity:
    storage: 8Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: rbdsc
  rbd:
      monitors:
        - '192.168.122.120:6789,192.168.122.121:6789,192.168.122.122:6789'
      pool: ssd-pool
      image: rbdpd
      fsType: ext4
      readOnly: false
      user: admin
      keyring: /etc/ceph/ceph.client.admin.keyring
```
这里创建连个PV，一个5G，一个8G。创建一个要求8G的pvc，它会选择8G的PV bond。当然最好使用pvc中的select中的label标识来选取PV。
```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: rbdclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 8Gi
  storageClassName: rbdsc
```
```
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: rbdsc
provisioner: kubernetes.io/rbd
parameters:
  monitors: 192.168.122.120:6789,192.168.122.121:6789,192.168.122.122:6789
  adminId: admin
  adminSecretName: ceph-secret
  adminSecretNamespace: default
  pool: ssd-pool
  userId: admin
  userSecretName: ceph-secret-user
  fsType: ext4
  imageFormat: "2"
  imageFeatures: "layering"
```
这里的storageclass支持imageFormat和imageFeatures关键字
```
apiVersion: v1
kind: Pod
metadata:
  name: rbd
spec:
  containers:
    - image: 192.168.122.1:5000/kubernetes/pause:v1
      name: rbd-rw
      volumeMounts:
      - name: rbdpd
        mountPath: /mnt/rbd
  volumes:
    - name: rbdpd
      persistentVolumeClaim:
        claimName: rbdclaim

```
具体参数的解析可以去官网查看
https://kubernetes.io/docs/concepts/storage/persistent-volumes/
```
[root@k8s-master rbd]# kubectl get pods
NAME                           READY     STATUS    RESTARTS   AGE
cheddar-2695380518-hkx69       1/1       Running   4          22d
cheddar-2695380518-p82xm       1/1       Running   4          22d
rbd                            1/1       Running   0          1h
stilton-2773127498-jbvtx       1/1       Running   4          22d
stilton-2773127498-qm37l       1/1       Running   4          22d
wensleydale-1811307530-djn5s   1/1       Running   4          22d
wensleydale-1811307530-kj1zk   1/1       Running   4          22d
[root@k8s-master rbd]# kubectl get pvc
NAME       STATUS    VOLUME    CAPACITY   ACCESSMODES   STORAGECLASS   AGE
rbdclaim   Bound     rbdpv8g   8Gi        RWO           rbdsc          1h
[root@k8s-master rbd]# kubectl get pv
NAME      CAPACITY   ACCESSMODES   RECLAIMPOLICY   STATUS      CLAIM              STORAGECLASS   REASON    AGE
rbdpv     5Gi        RWO           Recycle         Available                      rbdsc                    54m
rbdpv8g   8Gi        RWO           Recycle         Bound       default/rbdclaim   rbdsc                    48m
[root@k8s-master rbd]# kubectl get sc
NAME      TYPE
rbdsc     kubernetes.io/rbd 
```
在运行pod的节点上使用mount和docker inspect查看容器
```
[root@k8s-slave2 ~]# mount | grep rbd
/dev/rbd0 on /var/lib/kubelet/plugins/kubernetes.io/rbd/rbd/ssd-pool-image-rbdpd type ext4 (rw,relatime,seclabel,stripe=1024,data=ordered)
/dev/rbd0 on /var/lib/kubelet/pods/4f5249be-be12-11e7-8a16-525400db31fc/volumes/kubernetes.io~rbd/rbdpv8g type ext4 (rw,relatime,seclabel,stripe=1024,data=ordered)

[root@k8s-slave2 ~]# docker inspect e4d3d5e8dcd8
...
...
...
"Mounts": [
            {
                "Type": "bind",
                "Source": "/var/lib/kubelet/pods/4f5249be-be12-11e7-8a16-525400db31fc/volumes/kubernetes.io~rbd/rbdpv8g",
                "Destination": "/mnt/rbd",
                "Mode": "Z",
                "RW": true,
                "Propagation": "rprivate"
            },
...
...
...
```